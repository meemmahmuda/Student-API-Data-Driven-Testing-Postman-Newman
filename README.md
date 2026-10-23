# Student API Data Driven Testing (Postman + Newman)

Data Driven Test (DDT) automation for the **Student Management API** (`https://thetestingworldapi.com`). The same Postman collection is executed multiple times, once for each row of an external data file, using **Newman** from the command line. An HTML report is generated with **newman-reporter-htmlextra**.

---

## Latest Test Run Summary

| Metric | Result |
| --- | --- |
| Run date | 23 October 2026 |
| Collection | Student_DDT |
| Environment | Student_DDT |
| Iterations (data rows) | 3 |
| Requests per iteration | 4 |
| Total requests | 12 |
| Total assertions | 105 |
| Passed assertions | 99 |
| Failed assertions | 6 |
| Skipped tests | 0 |
| Total run duration | 2.8 s |
| Average response time | 147 ms |
| Total data received | 717 B |

All 12 requests were executed. The 6 failures are the same status code check repeated across the 3 iterations (details below).

---

## What Is Data Driven Testing?

Instead of hard-coding one set of student data in the request body, the collection reads values from an external data file (CSV or JSON). Newman runs the whole collection once per row, so a single collection validates the API with many different inputs.

In this run, 3 data rows produced 3 iterations, and each iteration created a different student.

---

## Test Flow (per iteration)

| # | Request | Method | Endpoint |
| --- | --- | --- | --- |
| 1 | Create Student | `POST` | `/api/studentsDetails` |
| 2 | Update Student | `PUT` | `/api/studentsDetails/{id}` |
| 3 | Create Technical Skills | `POST` | `/api/technicalskills` |
| 4 | Create Student Address | `POST` | `/api/addresses/{id}` |

The `id` returned by **Create Student** is saved to the environment and reused by the next requests in the same iteration.

---

## Test Data Used

| Iteration | Created Student | Updated To | Student ID |
| --- | --- | --- | --- |
| 1 | Mr. Mark Zukurburg (1987-05-23) | Mr. Helal Uddin (1987-05-23) | 10894416 |
| 2 | Ms. Adam Cruz (1990-05-23) | Ms. Samia Jaman (1990-05-23) | 10894417 |
| 3 | Mahmuda Binte Sayeed (1998-05-23) | Mahmuda Binte Sayeed (1998-05-23) | 10894418 |

Technical skills and address requests also send data such as languages, years of experience, last used year, house number, city, state, country and phone numbers.

---

## Results Per Iteration

| Iteration | Create Student | Update Student | Create Technical Skills | Create Student Address |
| --- | --- | --- | --- | --- |
| 1 | Passed | Passed | 5 of 6 passed | 5 of 6 passed |
| 2 | Passed | Passed | 5 of 6 passed | 5 of 6 passed |
| 3 | Passed | Passed | 5 of 6 passed | 5 of 6 passed |

Each iteration runs 35 assertions: 17 for Create Student, 6 for Update Student, 6 for Create Technical Skills and 6 for Create Student Address.

---

## What Is Being Tested

- **Status code**: 201 for create, 200 or 403 for update
- **Performance**: response time and response size thresholds
- **Data correctness**: the response values match the values sent from the data file
- **Data types**: names and date of birth are strings, ID is a number, body is an object
- **Non-empty fields**: first name, middle name, last name and date of birth
- **Business response**: `status` is `true` and the success message is returned

---

## Known Failures

| Iteration | Request | Failed Test | Error |
| --- | --- | --- | --- |
| 1, 2, 3 | Create Technical Skills | Verify that Status code is 201 or not | expected response to have status code 201 but got 200 |
| 1, 2, 3 | Create Student Address | Verify that Status code is 201 or not | expected response to have status code 201 but got 200 |

**Analysis:** Both endpoints return `200 OK` instead of `201 Created`, while the response body (`status: true` and the success message) is correct. The records are created successfully, so this is a mismatch between the expected status code in the test and the actual API behavior. Because it happens in every iteration, it is consistent API behavior and not a data issue. Either update the assertion to accept `200`, or change the API to return `201`.

---

## Project Structure

```
.
├── collection/
│   └── Student_DDT.postman_collection.json
├── environment/
│   └── Student_DDT.postman_environment.json
├── data/
│   └── student_data.csv
├── reports/
│   └── Student_DDT-2026-10-23-12-13-42-786-0.html
└── README.md
```

> Adjust the folder and file names to match your repository.

---

## Prerequisites

- [Node.js](https://nodejs.org/) (LTS recommended)
- [Postman](https://www.postman.com/) (to create or edit the collection)
- Newman and the htmlextra reporter

```bash
npm install -g newman
npm install -g newman-reporter-htmlextra
```

Verify the installation:

```bash
newman --version
```

---

## How to Run the Tests

Run the collection with the environment and the data file, then generate the HTML report:

```bash
newman run collection/Student_DDT.postman_collection.json \
  -e environment/Student_DDT.postman_environment.json \
  -d data/student_data.csv \
  -r htmlextra \
  --reporter-htmlextra-export reports/
```

The `-d` option points to the data file. Newman runs one iteration for each row in it.

### Run with CLI output only

```bash
newman run collection/Student_DDT.postman_collection.json \
  -e environment/Student_DDT.postman_environment.json \
  -d data/student_data.csv
```

### Run only a fixed number of iterations

```bash
newman run collection/Student_DDT.postman_collection.json \
  -e environment/Student_DDT.postman_environment.json \
  -d data/student_data.csv \
  -n 2
```

---

## Data File Format

CSV example (column names must match the variables used in the collection):

```csv
first_name,middle_name,last_name,date_of_birth
Mr.,Mark,Zukurburg,1987-05-23
Ms.,Adam,Cruz,1990-05-23
Mahmuda,Binte,Sayeed,1998-05-23
```

In the request body, use the variable with double curly braces, for example `{{first_name}}`. In test scripts, read the value with `pm.iterationData.get("first_name")`.

---

## Viewing the Report

Open the generated file in any browser:

```
reports/Student_DDT-2026-10-23-12-13-42-786-0.html
```

The report includes:

- Summary dashboard (requests, assertions, failures, skipped tests, iterations)
- Timing and data statistics
- List of failed tests with error messages
- Per-iteration and per-request details: method, URL, headers, request body, response body and assertion results
- Light and dark theme toggle

### Optional: Host the report with GitHub Pages

1. Copy the report to a `docs/` folder and rename it to `index.html`.
2. In your repository go to **Settings > Pages**.
3. Select the branch and the `/docs` folder, then save.
4. The report will be available at `https://<username>.github.io/<repository>/`.

---

## Tech Stack

| Tool | Purpose |
| --- | --- |
| Postman | API request design, test scripts and data variables |
| Newman | Command-line collection runner with data file support |
| newman-reporter-htmlextra | HTML report generation |
| Node.js | Runtime for Newman |

---

## Notes

- The API under test is a public practice API, so response times may vary and the data may be shared with other users.
- Response size and time thresholds in the assertions may need adjusting depending on the network and server load.

---

## Author

**Your Name**
GitHub: [@your-username](https://github.com/your-username)
