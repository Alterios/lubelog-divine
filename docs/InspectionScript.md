# InspectionScript

This document covers the JavaScript enhancements in `inspectionrecord.js` that support the public inspection workflow.

## Features

- **Public Selector**: Displays a template selector filtered for public link generation.
- **QR Generation**: Retrieves the hashed URL from the server and generates a QR code pointing directly to the public form using the new `createQRForUrl` helper (defined in `shared.js`).

## Related Files

- [[InspectionRecordsUI]]
- [[PublicController]]
- [[00_Project_Index]]

## Implementation Code

```javascript
function showPublicInspectionQRCodeSelectorModal() {
    var vehicleId = GetVehicleId().vehicleId;
    $.get(`/Vehicle/GetInspectionRecordTemplatesByVehicleId?vehicleId=${vehicleId}&isPublic=true`, function (data) {
        if (data) {
            $("#inspectionRecordTemplateModalContent").html(data);
            $('#inspectionRecordTemplateModal').modal('show');
            clearModalContentOnHide($('#inspectionRecordTemplateModal'));
        }
    });
}
function generatePublicInspectionQRCode(templateId) {
    var vehicleId = GetVehicleId().vehicleId;
    $.get(`/Public/GetPublicInspectionHash?vehicleId=${vehicleId}&templateId=${templateId}`, function (data) {
        if (data.success) {
            hideInspectionRecordTemplateSelectorModal();
            let urlToRender = `${window.location.origin}${data.url}`;
            createQRForUrl(urlToRender);
        } else {
            errorToast(data.message);
        }
    });
}
```

### Shared.js Helper

The `createQRForUrl` was added to `shared.js` to create QR codes from absolute URLs, bypassing the default index parameter appending.

```javascript
function createQRForUrl(urlToRender) {
    let qr = qrcode(0, 'M');
    qr.addData(urlToRender);
    qr.make();
    let svgData = qr.createSvgTag();
    Swal.fire({
        title: "QR Code",
        html: `<div class='qr-container'>${svgData}</div>`,
        confirmButtonText: 'Download',
        showCancelButton: true
    }).then(function (result) {
        if (result.isConfirmed) {
            downloadQR();
        }
    });
}
```
