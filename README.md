# LubeLogger - Divine Fork

This is a customized fork of the original [LubeLogger](https://github.com/hargata/lubelog) project.

### Why this Fork?

This version was created to streamline vehicle maintenance for fleet operations, specifically focusing on enhanced accountability and specialized user configurations that align with organizational SSO/OAuth workflows.

### Key Custom Features & Fixes

* **Authenticated Public Inspections**: Modified the public inspection form to require authentication (`[Authorize]`). This ensures that every submitted inspection is tied to a specific driver/user in the system.
* **User Tracking**: Backend logic captures the authenticated user's ID and Name for all public form submissions, mapping them to event webhooks and personalized odometer tracking.
* **Optional Inspection Costs**: Added logic to verify if inspection costs are optional at the vehicle level, allowing for cleaner record entry when costs aren't applicable.
* **Form Data Binding Fixes**: Resolved a bug where unchecked checkboxes/radio buttons in the public form would misalign with the underlying inspection template, ensuring "Yes/No" results are recorded accurately.
* **Internal Documentation**: Integrated custom project documentation directly into the repository for easier maintenance.

---

![image](https://github.com/hargata/lubelog/assets/155338622/545debcd-d80a-44da-b892-4c652ab0384a)

Self-Hosted, Open-Source, Web-Based Vehicle Maintenance and Fuel Mileage Tracker

Website: <https://lubelogger.com>

## Why

Because nobody should have to deal with a homemade spreadsheet or a shoebox full of receipts when it comes to vehicle maintenance.

## Showcase

[Promotional Brochure](https://lubelogger.com/brochure.pdf)

[Screenshots](/docs/screenshots.md)

## Demo

Try it out before you download it! The live demo resets every 20 minutes.

[Live Demo](https://demo.lubelogger.com) Login using username "test" and password "1234"

## Download

LubeLogger is available as both a Docker Image and a Windows Standalone Executable.

Read this [Getting Started Guide](https://docs.lubelogger.com/Installation/Getting%20Started) on how to download either of them

### Kubernetes Deployment

[Helm Chart](https://artifacthub.io/packages/helm/anza-labs/lubelogger) provided by [Anza-Labs](https://github.com/anza-labs)

### Need Help?

[Documentation](https://docs.lubelogger.com/)

[Troubleshooting Guide](https://docs.lubelogger.com/Installation/Troubleshooting)

[Search Existing Issues](https://github.com/hargata/lubelog/issues)

## Dependencies

- [Bootstrap](https://github.com/twbs/bootstrap)
* [LiteDB](https://github.com/mbdavid/litedb)
* [Npgsql](https://github.com/npgsql/npgsql)
* [Bootstrap-DatePicker](https://github.com/uxsolutions/bootstrap-datepicker)
* [SweetAlert2](https://github.com/sweetalert2/sweetalert2)
* [CsvHelper](https://github.com/JoshClose/CsvHelper)
* [Chart.js](https://github.com/chartjs/Chart.js)
* [Drawdown](https://github.com/adamvleggett/drawdown)
* [MailKit](https://github.com/jstedfast/MailKit)
* [Masonry](https://github.com/desandro/masonry)
* [QRCode-Generator](https://github.com/kazuhikoarase/qrcode-generator)

## License

MIT

## Support

Support this project by [Subscribing on Patreon](https://patreon.com/LubeLogger) or [Making a Donation](https://buy.stripe.com/aEU9Egc8DdMc9bO144)
