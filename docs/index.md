# InvoiceBldr

_Last Revision: 5/18/20_

_Latest Application Release: v1.5.1_

> The following documentation serves as a technical guide for InvoiceBldr, a fullstack application that allows <strong>Admin</strong> users to create invoices based on `.csv` datasets. The application supplies multiple pre-made `.csv` templates with pre-defined column names. These templates are based on two different overarching use-cases: <strong>service</strong>, and <strong>manufactoring</strong>.

> Invoices are stored in a PostgreSQL database and made accessible to <strong>Client</strong> user accounts. Clients can view details regarding each invoice, download attachments and supporting documents, leave notes to communicate back to the Admin, and change the status of each invoice from <strong>pending</strong> to <strong>disputed</strong> and finally <strong>approved</strong>.

> InvoiceBldr integrates Google resources such as Calendar, Sheets, and Mail using OAuth2.0 and the Google Client Library. Google API integration streamlines communication from within the app, and dynamically updates both Admin and Client calendars to reflect invoice due dates.

> Admins can extract aging reports for all or a subset of their associated Clients. Reports are fully customizable, allowing users to select and filter fields to extract.
