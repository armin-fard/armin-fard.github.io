# Developers

## Stack

- `Back-End`
    <ul>
    <li><a href="https://www.python.org/">Python</a></li>
    <li><a href="https://palletsprojects.com/p/flask/">Flask</a></li>
    </ul>
    </br>

- `Front-End`

  <ul>
  <li><a href="https://www.javascript.com/">JavaScript</a></li>
  <li><a href="https://reactjs.org/">React</a></li>
  <li><a href="https://redux.js.org/">Redux</a></li>
  <li><a href="https://ant.design/">Ant Design</a></li>
  </ul>

- `Database`
  <ul>
  <li><a href="https://www.postgres.jlmstrategic.com/">PostgreSQL</a></li>
  </ul>

## Installation/Set-up (Development)

    # Back-end
    pipenv install      # Initial installation
    pipenv shell        # Start the virtual environment
    python app.py       # Start the back-end

    # Front-end
    npm install         # Initial installation
    npm start           # Start the front-end

## Environment Description

### <u>Front-End</u>

> Front-end is built with React, utilizing redux for global state management/updates.

> Most of the components are leveraged from Ant Design, including forms, buttons, grids, and more.

> The front-end is currently hosted in the <a href="https://aws.amazon.com/s3">AWS S3</a> bucket.

### Back-End

> The back-end used to be hosted on an <a href="https://aws.amazon.com/ec2">AWS EC2</a> instance but recently has been transitioned into <a href="https://aws.amazon.com/elasticbeanstalk/">AWS EBS</a> (Elastic Beanstalk)

> Consequently, the zipped back-end code is stored in AWS S3 and is set-up by AWS EBS to compile the files and establish the server.

## File Structure

\*<i>This section will <strong>not</strong> list out every file in the directory. Instead, it will focus on files with relative importance and lay out the general foundation for its core structure.<i>

### Front-End (Client)

    App.js
    index.js
    store.js                              # File for setting up middleware, mainly redux
    src/
        actions/                          # Redux actions
            admin/                        # Admin-related action types
            profiles/                     # Profile-related action types
            projects/                     # Project-related action types

        components/                       # Folder for all the components.
            admin                         # Contains functions/logic for admin edit/view
            client                        # Contains functions/logic for  client dashboard
            common                        # Contains functions/logic for  error handler and private routes
            landingPage                   # Contains functions/logic for  landing page
            lazyComponents                # Contains functions/logic for  candidate dashboard

        images/                           # Contains the image assets used in the application
        reducers/                         # Contains all redux reducers
        utils/                            # Contains all utility files/helper functions

### <u>Back-End</u> (Server)

    app.py
    .env
    login.py
    authenticate.py                       # Contains all functions/logic for authenticating requests to the server
    helperFunctions.py                    # Contains all functions/logic for providing/destructuring helper functions
    .ebxtensions/
        install-wkhtmltopdf.config        # Contains all functions/logic for installing wkhtmltopdf when deploying to EBS

    api\endpoints/
        adminlogs.py                      # Contains all functions/logic for establishing/saving admin activity logs
        assign.py                         # Contains all functions/logic for candidate/project association
        delete.py                         # Contains all functions/logic for deleting candidates
        edit.py                           # Contains all functions/logic for editing candidate information
        email.py                          # Contains all functions/logic for sending emails within the app
        export.py                         # Contains all functions/logic for exporting database values to csv
        interview.py                      # Contains all functions/logic for modifying candidate interviews
        note.py                           # Contains all functions/logic for candidate notes
        notifications.py                  # Contains all functions/logic for admin notifications
        pdf.py                            # Contains all functions/logic for rendering candidate profiles in pdf
        profiles.py                       # Contains all functions/logic for profiles CRUD
        projects.py                       # Contains all functions/logic for projects CRUD
        submit.py                         # Contains all functions/logic for admin/candidate creation

        client/
            clientlogs.py                 # Contains all functions/logic for establishing/saving client activity logs
            deleted.py                    # Contains all functions/logic for  deleting projects/orders
            editProfile.py                # Contains all functions/logic for editing client profile
            hired.py                      # Contains all functions/logic for managing hired candidates
            order.py                      # Contains all functions/logic for modifying order information
            pending.py                    # Contains all functions/logic for managing pending candidates
            requested.py                  # Contains all functions/logic for managing requested candidates
            resume.py                     # Contains all functions/logic for modifying candidate resumes
            saved.py                      # Contains all functions/logic for managing saved candidates
            scheduled.py                  # Contains all functions/logic for managing scheduled interviews/candidates

    cronJobs/                             # Contains all functions/logic for time-based operations
    templates/                            # Folder containing all templates rendered within the app
        emails/

## Notable Features & Functionalities

> This section will highlight the major features and functionalities within TeamBldr.

### Logs

#### Method:

> Most actions triggered in TeamBldr go through a helper function that prepares and organizes the data to save as a log.

#### Purpose:

> Tracking all activities within the application can be extremely beneficial. The usage information can be extracted via API call or csv export, transformed via technical script or Tableau, and loaded onto a dashboard or online Tableau interface.

> Based on the result of the ETL flow, analysts can derive meaning from the activity within TeamBldr. For instance, there are statistical models and methods that can employed by examining the client activity within the application and tying it back to the likelihood of that client's willingness to hire.

> Given the client's tendency towards certain logged actions within Teambldr over others, we can run an analysis on the impact of those logged actions (i.e. profile viewed, interview scheduled) and put together a basic classification model to divide the clients into groups. Once there is a categorization for the client's willingness to hire based on their recorded logged actions, we can run a logistic regression for any client that we want to classify as having high willingness to hire or not.

### Matching Algorithm

> Matching Criteria

| Criterion               | Importance(Low to High) | Weight (Out of 100) |
| ----------------------- | :---------------------: | :-----------------: |
| Position                |          High           |         13          |
| Location/Relocation     |          High           |         13          |
| Salary Expectation      |          High           |         13          |
| Company Experience      |          High           |         13          |
| Project Experience      |          High           |         13          |
| Required Skills         |          High           |         13          |
| Required Certification  |         Medium          |         7.5         |
| Years of Experience     |         Medium          |         7.5         |
| Preferred Certification |           Low           |         3.5         |
| Preferred Skills        |           Low           |         3.5         |
| Work Experience\*       |           N/A           |         N/A         |

\* Work experience matching compares all of the candidate's work experience with the order job description. Based on a match from cosine similarity, any matching words append to the overall score of the candidate's matching.

### PDF Render

> wkhtmltopdf

### CSV Export

> Integrated SQL queries

## Database

> PostgreSQL is our database, hosted on AWS EC2.

### Basic Tables & Columns

> For information regarding the data types in PostgreSQL, please consult this <a href="https://www.postgresqltutorial.com/postgresql-data-types/">link</a>.

| Table   | Join Table         | Table     |
| ------- | ------------------ | --------- |
| `users` | `project_profiles` | `project` |

### Relational Table & Columns

| Table   | Join Table         | Table     |
| ------- | ------------------ | --------- |
| `users` | `project_profiles` | `project` |

#### project_profiles:

|         Column          |     Type      | Description                                                 |
| :---------------------: | :-----------: | ----------------------------------------------------------- |
| `join_id(primary key)`  |  `serial ID`  | `ID depicting the join between candidates and projects`     |
|      `bullhorn_id`      |   `integer`   | `Old ID reference to the candidates migrated from Bullhorn` |
|     `candidate_id`      |    `UUID`     | `New, official ID reference for the candidates`             |
|      `project_id`       |    `UUID`     | `Official ID reference for the associated project`          |
|     `project_name`      | `varchar(50)` | `Name of the project`                                       |
| `candidate_first_name*` | `varchar(50)` | `First name of the candidate associated`                    |
| `candidate_last_name*`  | `varchar(50)` | `First name of the candidate associated.`                   |
|      `date_added`       |    `date`     | `Date in which the candidate was assigned to the project`   |
|   `project_position`    | `varchar(50)` | `Name of the candidate's position displayed to the project` |
|       `bill_rate`       |    `real`     | `Bill rate(pay rate * qualifier) displayed to the client`   |
| `bill_rate_visibility`  |   `boolean`   | `Client's visibility determinant for candidate's bill rate` |
|       `order_id`        |    `UUID`     | `Official ID reference for the associated order, if any`    |
|      `order_name`       | `varchar(60)` | `Name of the associated order, if any`                      |
|         `stage`         | `varchar(50)` | `Candidate's current stage in the hiring pipeline`          |

\*The main reason for the separation between first and last name is to ensure the clients to go behind our back and poach our candidates.
