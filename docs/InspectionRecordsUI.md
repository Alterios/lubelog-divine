# InspectionRecordsUI

This document covers the UI enhancements made to the Inspection Records tab to support public QR code generation.

## Features

- **Public Inspection Entry**: Added a new option to the inspection records dropdown.
- **Improved Workflow**: Users can now trigger a template selector specifically for generating public links.

## Related Files

- [[InspectionScript]]
- [[00_Project_Index]]

## Implementation Details

The following button was added to the records dropdown:

```html
<li><a class="dropdown-item" href="#" onclick="showPublicInspectionQRCodeSelectorModal()">@translator.Translate(userLanguage, "Public Inspection QR Code")</a></li>
```

This triggers a modified template selector that, when a template is chosen, calls the hashing logic instead of the internal entry logic.

```
