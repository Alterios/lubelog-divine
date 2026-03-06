# PublicController

The `PublicController` handles unauthenticated requests for the vehicle inspection form. It uses hashed identifiers for vehicles and templates to prevent enumeration and provide a secure, publicly accessible link.

## Key Features

- **Hashed Links**: Obfuscates vehicle and template IDs in the URL.
- **Simplified Access**: Provides a streamlined, clean web form for inspections, but now requires `[Authorize]` so that the submitting driver's identity can be tracked via SSO.
- **Integration**: Adapts the internal `SaveInspectionRecord` logic to support form submissions, mapping the authenticated user's ID to odometer auto-insertions, plan record creations, and event webhooks.

## Related Files

- [[00_Project_Index]]
- [[InspectionView]]
- [[FinishedView]]

## Implementation Details

The controller retrieves the vehicle and template using a SHA256 hash of the ID with a "LubeLoggerPublic_" prefix. This ensures that even if a link is leaked, users cannot guess other vehicle or template IDs.

```csharp
using CarCareTracker.External.Interfaces;
using CarCareTracker.Helper;
using CarCareTracker.Logic;
using CarCareTracker.Models;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

namespace CarCareTracker.Controllers
{
    [Authorize]
    public class PublicController : Controller
    {
        private int GetUserID()
        {
            return int.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier) ?? "0");
        }
        private readonly IVehicleDataAccess _vehicleDataAccess;
        private readonly IInspectionRecordDataAccess _inspectionRecordDataAccess;
        private readonly IInspectionRecordTemplateDataAccess _inspectionRecordTemplateDataAccess;
        private readonly IFileHelper _fileHelper;
        private readonly IConfigHelper _config;
        private readonly ILogger<PublicController> _logger;
        private readonly IOdometerLogic _odometerLogic;
        private readonly IServiceRecordDataAccess _serviceRecordDataAccess;
        private readonly IPlanRecordDataAccess _planRecordDataAccess;
        private readonly IEventLogic _eventLogic;

        public PublicController(IVehicleDataAccess vehicleDataAccess,
            IInspectionRecordDataAccess inspectionRecordDataAccess,
            IInspectionRecordTemplateDataAccess inspectionRecordTemplateDataAccess,
            IFileHelper fileHelper,
            IConfigHelper config,
            ILogger<PublicController> logger,
            IOdometerLogic odometerLogic,
            IServiceRecordDataAccess serviceRecordDataAccess,
            IPlanRecordDataAccess planRecordDataAccess,
            IEventLogic eventLogic)
        {
            _vehicleDataAccess = vehicleDataAccess;
            _inspectionRecordDataAccess = inspectionRecordDataAccess;
            _inspectionRecordTemplateDataAccess = inspectionRecordTemplateDataAccess;
            _fileHelper = fileHelper;
            _config = config;
            _logger = logger;
            _odometerLogic = odometerLogic;
            _serviceRecordDataAccess = serviceRecordDataAccess;
            _planRecordDataAccess = planRecordDataAccess;
            _eventLogic = eventLogic;
        }

        private string GetHash(int id)
        {
            return StaticHelper.GetHash($"LubeLoggerPublic_{id}");
        }

        public IActionResult Inspection(string v, string t)
        {
            try
            {
                // Find vehicle by hash
                var vehicles = _vehicleDataAccess.GetVehicles();
                var vehicle = vehicles.FirstOrDefault(x => GetHash(x.Id) == v);
                if (vehicle == null)
                {
                    return NotFound("Vehicle not found or invalid link.");
                }

                // Find template by hash
                var templates = _inspectionRecordTemplateDataAccess.GetInspectionRecordTemplatesByVehicleId(vehicle.Id);
                var template = templates.FirstOrDefault(x => GetHash(x.Id) == t);
                if (template == null)
                {
                    return NotFound("Inspection template not found or invalid link.");
                }

                // Get the full template with fields
                var templateWithFields = _inspectionRecordTemplateDataAccess.GetInspectionRecordTemplateById(template.Id);

                var viewModel = new InspectionRecordInput
                {
                    VehicleId = vehicle.Id,
                    Description = template.Description,
                    Fields = templateWithFields.Fields
                };

                ViewBag.VehicleName = $"{vehicle.Year} {vehicle.Make} {vehicle.Model}";
                ViewBag.VehicleHash = v;
                ViewBag.TemplateHash = t;

                return View(viewModel);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error loading public inspection form.");
                return View("Error");
            }
        }

        [HttpPost]
        public IActionResult SubmitInspection(InspectionRecordInput inspectionRecord)
        {
            try
            {
                if (inspectionRecord.VehicleId == default)
                {
                    return Json(new OperationResponse { Success = false, Message = "Invalid Vehicle ID" });
                }

                // Save the inspection record
                var convertedRecord = inspectionRecord.ToInspectionRecord();
                var result = _inspectionRecordDataAccess.SaveInspectionRecordToVehicle(convertedRecord);

                if (result)
                {
                    // Handle Odometer/Plan Record auto-insertion if enabled
                    var userConfig = _config.GetUserConfig(User); // Get authenticated user's config
                    
                    if (userConfig.EnableAutoOdometerInsert)
                    {
                        var odometerRecord = new OdometerRecord
                        {
                            VehicleId = inspectionRecord.VehicleId,
                            Date = DateTime.Parse(inspectionRecord.Date),
                            Mileage = inspectionRecord.Mileage,
                            Notes = $"Auto-inserted via Inspection: {inspectionRecord.Description}"
                        };
                        _odometerLogic.AutoInsertOdometerRecord(odometerRecord);
                    }

                    // Create action items for failed markers
                    foreach (var field in inspectionRecord.Fields)
                    {
                        if (field.ToInspectionRecordResult().Failed)
                        {
                            var planRecord = new PlanRecord
                            {
                                VehicleId = inspectionRecord.VehicleId,
                                DateCreated = DateTime.Now,
                                DateModified = DateTime.Now,
                                Description = $"FAILED INSPECTION: {field.Description}",
                                Notes = $"This item failed during the daily inspection: {inspectionRecord.Description}",
                                Priority = PlanPriority.Critical,
                                Progress = PlanProgress.Backlog,
                                ImportMode = ImportMode.PlanRecord
                            };
                            _planRecordDataAccess.SavePlanRecordToVehicle(planRecord);
                        }
                    }

                    _eventLogic.PublishEvent(GetUserID(), WebHookPayload.FromInspectionRecord(convertedRecord, "inspectionrecord.add", User.Identity?.Name ?? string.Empty));
                    return Json(new OperationResponse { Success = true, Message = "Inspection submitted successfully." });
                }

                return Json(new OperationResponse { Success = false, Message = StaticHelper.GenericErrorMessage });
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error submitting public inspection.");
                return Json(new OperationResponse { Success = false, Message = ex.Message });
            }
        }

        [HttpGet]
        public IActionResult GetPublicInspectionHash(int vehicleId, int templateId)
        {
            try
            {
                var vHash = GetHash(vehicleId);
                var tHash = GetHash(templateId);
                var url = $"{_config.GetServerDomain()}/Public/Inspection?v={vHash}&t={tHash}";
                return Json(new { success = true, url = url });
            }
            catch (Exception ex)
            {
                return Json(new { success = false, message = ex.Message });
            }
        }

        public IActionResult Finished()
        {
            return View();
        }
    }
}
```
