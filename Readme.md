# HR System API - MuleSoft Project

This project implements a RESTful API for managing staff entities within the HR system using MuleSoft. The API provides endpoints for CRUD operations for staff within HR application. Additionally, it supports notifications of changed staff records and requires special privileges for certain operations.

## Assmuptions

1. HR System uses a tradaitional RDBMS i.e MySQL to store staff information
2. All the the clietns intending to access the HR system API's should get themselves registerd first. It's a manual registartion where they would provided access keys and assigned a role within the HR System. The roles are READ, WRITE and ADMIN with the follwong permissions
   - READ - Allowed to fetch staff information
   - WRITE - Allowed to Create, Delete & Modify Staff information
   - ADMIN - All permissions

Other options like LDAP & OAuth could be used to store and restrict client access within the HR system, but to simplify the development and testing I have decoded to store the role information within a daatbase. However, the concept remains the same i.e user authorization

## Getting Started

To run the MuleSoft project locally, follow these steps:

1. Clone this repository to your local machine.
2. Open the project in Anypoint Studio.
3. Configure the project to connect to your MuleSoft runtime environment.
4. Run the project within Anypoint Studio or deploy it to your MuleSoft runtime environment.

## Endpoints

The API exposes the following endpoints:

- **GET /api/staff**: Retrieve a list of all staff members.
- **POST /api/staff**: Create a new staff member.
- **PUT /api/staff/{staffId}**: Update an existing staff member.
- **DELETE /api/staff/{staffId}**: Offboard a staff member.
- **POST /api/notifications/staffChange**: Notify the API of changed staff records.

## Security

- Authentication: Managed through ClientID enforcement policy within Cloudhub. All the API request must send `client_id` and `client_secret` headers.
- Authorization: Privilege verification is performed using checking the role assigned to client credentials i.e `client_id` and `client_secret` passed during the api call .

## Use Case 1: Periodic Consumption

Clients can periodically consume the API to maintain a copy of the staff records by making GET requests to the `/api/staff` endpoint.

1. `/api/staff` lists all the staff depending upon the query criterion specified in the query paramaeters. The query parameters are
   - first_name
   - last_name
   - role
   - active/inactive flag
   - ps (_page size i.e number of records to be returned during pagiantion default = 50_)
   - p (_page number default = 0_)

Pagination is implemeted to support fetching multiple records. Please consult the response provided below for reference

```
curl --location 'https://hr.tafensw.gov.au/wms-exp/api/staff?active=true&p=2&ps10 \
    --header 'client_id: XXXXXXXXXXXXXXXX' \
    --header 'client_secret: XXXXXXXXXXXXX'

{
    "total": 324,
    "staff":[
        {
            "id": "1",
            "firstName": "John",
            "lastName": "Doe",
            "email": "john.doe@example.com",
            "role": "Manager",
            "active": true
        },
        {
            "id": "2",
            "firstName": "Jane",
            "lastName": "Citizen",
            "email": "jane.citizen@example.com",
            "role": "Employee",
            "active": true
        }
    ]
}
```

2. `/api/staff/{staffId}` - HTTP GET  
   Return details for a specific staff based on staffId. Please consult the response provided below for reference

```
curl --location 'https://hr.tafensw.gov.au/wms-exp/api/staff/1 \
    --header 'client_id: XXXXXXXXXXXXXXXX' \
    --header 'client_secret: XXXXXXXXXXXXX'

    {

         "id": "1",
         "firstName": "John",
         "lastName": "Doe",
         "email": "john.doe@example.com",
         "role": "Manager",
         "active": true
    }
```

3. `/api/staff/{staffId}` - HTTP POST  
   Create a staff in HR System and return it's ID. Please consult the response provided below for reference

```
curl --location 'https://hr.tafensw.gov.au/wms-exp/api/staff/1 \
    --header 'client_id: XXXXXXXXXXXXXXXX' \
    --header 'client_secret: XXXXXXXXXXXXX'

    {
         "firstName": "John",
         "lastName": "Doe",
         "email": "john.doe@example.com",
         "role": "Manager",
         "active": true
    }

Reponse:
    {
        "id": 1078
    }

```

4. `/api/staff/{staffId}` - HTTP PUT  
   Update a staff details for a specific staff based on staffId. Please consult the request provided below for reference

```
    curl --request PUT --location 'https://hr.tafensw.gov.au/wms-exp/api/staff/1 \
        --header 'client_id: XXXXXXXXXXXXXXXX' \
        --header 'client_secret: XXXXXXXXXXXXX'
        --data-raw '{
                "role": "Sr. manager"
                }'
```

5. `/api/staff/{staffId}` - HTTP DELETE  
   Offboard a staff member. Please consult the request provided below for reference
   Please note the as part of Offboarding only the active flag for the staff is set to false. The Staff isn't physically deleted from HR database

```
    curl --request DELETE --location 'https://hr.tafensw.gov.au/wms-exp/api/staff/1 \
        --header 'client_id: XXXXXXXXXXXXXXXX' \
        --header 'client_secret: XXXXXXXXXXXXX'

```

## Use Case 2: Notification of Changed Records

The API provides an endpoint (`/api/notification`) for the HR system to notify the API of changed staff records.

1. `/api/notification/staff` - HTTP POST  
   Notify of a nrewly created staff . The Staff is subsequently created in the procurement system. Please consult the request provided below for reference

```
    curl --request PUT --location 'https://hr.tafensw.gov.au/wms-exp/api/staff/1 \
        --header 'client_id: XXXXXXXXXXXXXXXX' \
        --header 'client_secret: XXXXXXXXXXXXX'
        --data-raw ' {
            "id": "1",
            "firstName": "John",
            "lastName": "Doe",
            "email": "john.doe@example.com",
            "role": "Manager",
            "active": true
        }'
```

2. `/api/notification/staff{staffId}` - HTTP PUT  
   Update staff details for a specific staff based on staffId. Please consult the request provided below for reference

   **Please note in the example below deletion is defined setting active attribute of the staff to false**

```
    curl --request PUT --location 'https://hr.tafensw.gov.au/wms-exp/api/staff/1 \
        --header 'client_id: XXXXXXXXXXXXXXXX' \
        --header 'client_secret: XXXXXXXXXXXXX'
        --data-raw '{
                "active": false
                }'

```

## Use Case 3: Security for Creation and Removal of Staff

Please refer to [Assumptions](#assmuptions) and [Security](#security) sections for more details . Only authorized client with READ, WRITE & ADMIN role can perform API operations on HR API.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
