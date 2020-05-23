# InvoiceBldr

_Last Revision: 5/18/20_

_Latest Application Release: v1.5.1_

> The following documentation serves as a guide and technical documentation for InvoiceBldr, a sample application that allows <strong>Admin</strong> users to create invoices based on .csv datasets. The application supplies multiple pre-made .csv templates with pre-defined column names. These templates are based on two different overarching business types: <strong>service</strong>, and <strong>manufactoring</strong>.

> Invoices are stored in a PostgreSQL database and made accessible to <strong>Client</strong> users on the front-end UI application via password protected, secure accounts . Clients can view details regarding each invoice, download attachments and supporting documents, leave notes to communicate back to the Admin, and finally approve each invoice to be processed by Admins.

> InvoiceBldr integrates Google services such as Calendar, Sheets, and Gmail using OAuth2.0 and the Google Client Library. The integration streamlines email communication from within the app, and dynamically updates both Admins' and Clients' Calendar to reflect invoice due dates.

> Admins can extract aging reports for all or a subset of their associated Clients. Reports are fully customizable, allowing users to select and filter fields to extract.
