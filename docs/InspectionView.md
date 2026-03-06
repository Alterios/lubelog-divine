# InspectionView

The `Inspection.cshtml` view provides a mobile-friendly interface for vehicle inspections. It is designed to be premium, responsive, and easy to use on small screens.

## Features

- **Responsive Layout**: Uses a single-column card layout optimized for mobile devices.
- **Dynamic Field Rendering**: Supports Radio, Check, and Text inspection fields.
- **Auto-validation**: Ensures required fields like odometer/mileage are filled.
- **AJAX Submission**: Uses jQuery to submit data to the `PublicController` without page reloads.

## Related Files

- [[PublicController]]
- [[00_Project_Index]]

## Implementation Code

```razor
@using CarCareTracker.Models
@model InspectionRecordInput
@{
    Layout = "_Layout";
    ViewData["Title"] = "Vehicle Inspection";
    var vehicleName = ViewBag.VehicleName as string;
}
@section Scripts {
    <script>
        function showLoader() {
            var btn = $('.btn-submit');
            btn.prop('disabled', true);
            btn.html('<span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span> Submitting...');
        }
        function hideLoader() {
            var btn = $('.btn-submit');
            btn.prop('disabled', false);
            btn.html('Submit Inspection');
        }
        function submitInspection() {
            var formData = $('#inspectionForm').serialize();
            showLoader();
            $.post('/Public/SubmitInspection', formData, function(data) {
                hideLoader();
                if (data.success) {
                    window.location.href = '/Public/Finished';
                } else {
                    Swal.fire({
                        icon: 'error',
                        title: 'Error',
                        text: data.message
                    });
                }
            });
        }
    </script>
}
<style>
    body {
        background-color: var(--bs-body-bg);
    }
    .inspection-card {
        border-radius: 15px;
        box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        margin-top: 20px;
        margin-bottom: 20px;
        border: none;
    }
    .inspection-header {
        background: linear-gradient(135deg, #0d6efd 0%, #0a58ca 100%);
        color: white;
        border-radius: 15px 15px 0 0;
        padding: 20px;
        text-align: center;
    }
    .vehicle-info {
        font-size: 1.2rem;
        font-weight: bold;
        margin-bottom: 5px;
    }
    .template-info {
        font-size: 0.9rem;
        opacity: 0.9;
    }
    .form-group-custom {
        margin-bottom: 20px;
        padding: 15px;
        border-bottom: 1px solid var(--bs-border-color);
    }
    .form-group-custom:last-child {
        border-bottom: none;
    }
    .field-name {
        font-weight: 600;
        margin-bottom: 10px;
        display: block;
    }
    .option-container {
        display: flex;
        flex-direction: column;
        gap: 10px;
    }
    .btn-submit {
        border-radius: 10px;
        padding: 12px;
        font-weight: bold;
        width: 100%;
        margin-top: 10px;
    }
    .lubelogger-body-container {
        max-width: 600px;
    }
    .failed-label {
        color: #dc3545;
        font-size: 0.8rem;
        font-style: italic;
    }
</style>

<div class="card inspection-card">
    <div class="inspection-header">
        <div class="vehicle-info">@vehicleName</div>
        <div class="template-info">@Model.Description</div>
    </div>
    <div class="card-body">
        <form id="inspectionForm">
            @Html.HiddenFor(m => m.VehicleId)
            @Html.HiddenFor(m => m.Description)
            
            <div class="mb-3">
                <label class="form-label font-weight-bold">Current Odometer</label>
                <input type="number" name="Mileage" class="form-control form-control-lg" placeholder="Enter mileage..." required />
            </div>

            <div class="mb-3">
                <label class="form-label font-weight-bold">Inspection Date</label>
                <input type="text" name="Date" class="form-control" value="@DateTime.Now.ToShortDateString()" readonly />
            </div>

            <hr />

            @for (int i = 0; i < Model.Fields.Count; i++)
            {
                <div class="form-group-custom">
                    <span class="field-name">@Model.Fields[i].Description</span>
                    @Html.HiddenFor(m => m.Fields[i].Description)
                    @Html.HiddenFor(m => m.Fields[i].FieldType)
                    
                    <div class="option-container">
                        @if (Model.Fields[i].FieldType == InspectionFieldType.Radio)
                        {
                            @for (int j = 0; j < Model.Fields[i].Options.Count; j++)
                            {
                                <div class="form-check">
                                    <input class="form-check-input" type="radio" 
                                           id="field_@(i)_opt_@(j)" 
                                           onchange="$(this).closest('.option-container').find('.hidden-radio').val('false'); $(this).siblings('.hidden-radio').val('true');">
                                    <input type="hidden" class="hidden-radio" name="Fields[@i].Options[@j].IsSelected" value="false" />
                                    <label class="form-check-label" for="field_@(i)_opt_@(j)">
                                        @Model.Fields[i].Options[j].Description
                                        @if (Model.Fields[i].Options[j].IsFail)
                                        {
                                            <span class="failed-label">(Fail)</span>
                                        }
                                    </label>
                                    <input type="hidden" name="Fields[@i].Options[@j].Description" value="@Model.Fields[i].Options[j].Description" />
                                    <input type="hidden" name="Fields[@i].Options[@j].IsFail" value="@(Model.Fields[i].Options[j].IsFail.ToString().ToLower())" />
                                </div>
                            }
                        }
                        else if (Model.Fields[i].FieldType == InspectionFieldType.Check)
                        {
                            @for (int j = 0; j < Model.Fields[i].Options.Count; j++)
                            {
                                <div class="form-check">
                                    <input class="form-check-input" type="checkbox" 
                                           id="field_@(i)_opt_@(j)" 
                                           onchange="$(this).siblings('.hidden-checkbox').val($(this).is(':checked').toString().toLowerCase());">
                                    <input type="hidden" class="hidden-checkbox" name="Fields[@i].Options[@j].IsSelected" value="false" />
                                    <label class="form-check-label" for="field_@(i)_opt_@(j)">
                                        @Model.Fields[i].Options[j].Description
                                        @if (Model.Fields[i].Options[j].IsFail)
                                        {
                                            <span class="failed-label">(Fail if unchecked)</span>
                                        }
                                    </label>
                                    <input type="hidden" name="Fields[@i].Options[@j].Description" value="@Model.Fields[i].Options[j].Description" />
                                    <input type="hidden" name="Fields[@i].Options[@j].IsFail" value="@(Model.Fields[i].Options[j].IsFail.ToString().ToLower())" />
                                </div>
                            }
                        }
                        else
                        {
                            <input type="text" name="Fields[@i].Value" class="form-control" placeholder="Enter details..." />
                        }

                        @if (Model.Fields[i].HasNotes)
                        {
                            <textarea name="Fields[@i].Notes" class="form-control mt-2" rows="2" placeholder="Notes..."></textarea>
                        }
                    </div>
                </div>
            }

            <div class="mt-4">
                <button type="button" class="btn btn-primary btn-submit" onclick="submitInspection()">Submit Inspection</button>
            </div>
        </form>
    </div>
</div>
```
