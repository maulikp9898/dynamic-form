# Dynamic Form Schema System Documentation

## Table of Contents
1. [Introduction](#1-introduction)
2. [JSON Schema Structure](#2-json-schema-structure)
3. [Schema Properties Explained](#3-schema-properties-explained)
4. [Example: Simple Form Schema](#4-example-simple-form-schema)
5. [Example: Form Response](#5-example-form-response)
6. [Data Source and Transformation](#6-data-source-and-transformation)
7. [Database Storage with MongoDB](#7-database-storage-with-mongodb)
8. [Best Practices](#8-best-practices)

## 1. Introduction

This documentation outlines a flexible and powerful JSON-based schema for creating dynamic forms. The system allows for the definition of various field types, validation rules, dependencies between fields, and the ability to populate fields from external data sources. We use MongoDB for storing form schemas and responses, providing a flexible and scalable solution.

## 2. JSON Schema Structure

The overall structure of the form schema is as follows:

```json
{
  "formId": "string",
  "title": "string",
  "fields": [
    {
      "id": "string",
      "type": "string",
      "label": "string",
      "placeholder": "string",
      "required": boolean,
      "validation": {
        "min": number,
        "max": number,
        "pattern": "string"
      },
      "options": [
        {
          "value": "string",
          "label": "string"
        }
      ],
      "dependsOn": [
        {
          "field": "string",
          "value": any,
          "operator": "string"
        }
      ],
      "dataSource": {
        "type": "string",
        "url": "string",
        "method": "string",
        "transformation": {
          "path": "string",
          "mapping": {
            "value": "string",
            "label": "string"
          }
        }
      }
    }
  ]
}
```

## 3. Schema Properties Explained

### Top-level Properties

- `formId`: A unique identifier for the form.
- `title`: The title of the form.
- `fields`: An array of field objects that define the structure of the form.

### Field Properties

- `id`: A unique identifier for the field.
- `type`: The type of field. Possible values are: "text", "number", "select", "multiselect", "radio", "checkbox", "textarea".
- `label`: The label displayed for the field.
- `placeholder`: Placeholder text for text inputs.
- `required`: A boolean indicating if the field is mandatory.
- `validation`: An object containing validation rules.
  - `min`: Minimum value for number inputs or minimum selections for multiselect.
  - `max`: Maximum value for number inputs or maximum selections for multiselect.
  - `pattern`: A regex pattern for validating text inputs.
- `options`: An array of option objects for select, multiselect, radio, and checkbox inputs.
  - `value`: The value of the option.
  - `label`: The label displayed for the option.
- `dependsOn`: An array of dependency objects that determine when this field should be displayed.
  - `field`: The id of the field this dependency is based on.
  - `value`: The value(s) of the dependent field that trigger this field.
  - `operator`: The comparison operator. Possible values are: "equals", "notEquals", "in", "notIn", "greaterThan", "lessThan".
- `dataSource`: An object defining an external data source for populating field options.

## 4. Example: Simple Form Schema

Here's an example of a simple form schema:

```json
{
  "formId": "user-registration",
  "title": "User Registration Form",
  "fields": [
    {
      "id": "full-name",
      "type": "text",
      "label": "Full Name",
      "placeholder": "Enter your full name",
      "required": true
    },
    {
      "id": "email",
      "type": "text",
      "label": "Email Address",
      "placeholder": "Enter your email",
      "required": true,
      "validation": {
        "pattern": "^[\\w-\\.]+@([\\w-]+\\.)+[\\w-]{2,4}$"
      }
    },
    {
      "id": "age",
      "type": "number",
      "label": "Age",
      "required": true,
      "validation": {
        "min": 18,
        "max": 120
      }
    },
    {
      "id": "country",
      "type": "select",
      "label": "Country",
      "required": true,
      "options": [
        { "value": "us", "label": "United States" },
        { "value": "ca", "label": "Canada" },
        { "value": "uk", "label": "United Kingdom" }
      ]
    },
    {
      "id": "terms-agreement",
      "type": "checkbox",
      "label": "I agree to the terms and conditions",
      "required": true
    }
  ]
}
```

## 5. Example: Form Response

When a user submits a form, the response might look like this:

```json
{
  "responseId": "resp_1234567890",
  "formId": "user-registration",
  "submittedAt": "2023-07-18T14:30:00Z",
  "data": {
    "full-name": "John Doe",
    "email": "john.doe@example.com",
    "age": 30,
    "country": "us",
    "terms-agreement": true
  }
}
```

## 6. Data Source and Transformation

The `dataSource` field allows you to populate options for select or multiselect fields from an external API. Here's a detailed explanation:

```json
"dataSource": {
  "type": "api",
  "url": "https://api.example.com/data",
  "method": "GET",
  "transformation": {
    "path": "$.items[*]",
    "mapping": {
      "value": "id",
      "label": "name"
    }
  }
}
```

- `type`: Currently supports "api" for fetching data from an API endpoint.
- `url`: The URL of the API endpoint.
- `method`: The HTTP method to use (GET or POST).
- `transformation`: Specifies how to transform the API response into options.
  - `path`: A JSONPath expression to select the relevant part of the API response.
  - `mapping`: Specifies which fields in the extracted data should be used for the `value` and `label` of each option.

For example, if the API returns:

```json
{
  "items": [
    { "id": 1, "name": "Option 1" },
    { "id": 2, "name": "Option 2" }
  ]
}
```

The transformation would result in options:

```json
[
  { "value": "1", "label": "Option 1" },
  { "value": "2", "label": "Option 2" }
]
```

## 7. Database Storage with MongoDB

We use MongoDB to store both form schemas and form responses. Here's how we structure our collections:

### Form Schemas Collection

```javascript
db.createCollection("forms");

db.forms.insertOne({
  formId: "user-registration",
  title: "User Registration Form",
  schema: {
    // The entire form schema JSON goes here
  },
  createdAt: new Date(),
  updatedAt: new Date()
});
```

### Form Responses Collection

```javascript
db.createCollection("form_responses");

db.form_responses.insertOne({
  responseId: "resp_1234567890",
  formId: "user-registration",
  submittedAt: new Date(),
  data: {
    // The form response data goes here
    "full-name": "John Doe",
    "email": "john.doe@example.com",
    "age": 30,
    // ... other fields
  }
});
```

## 8. Best Practices

1. **Indexing**: Create indexes on frequently queried fields to improve performance.
   ```javascript
   db.forms.createIndex({ formId: 1 });
   db.form_responses.createIndex({ formId: 1, submittedAt: 1 });
   ```

2. **Validation**: Use MongoDB schema validation to ensure data integrity.
   ```javascript
   db.createCollection("forms", {
     validator: {
       $jsonSchema: {
         bsonType: "object",
         required: ["formId", "title", "schema"],
         properties: {
           formId: {
             bsonType: "string",
             description: "must be a string and is required"
           },
           title: {
             bsonType: "string",
             description: "must be a string and is required"
           },
           schema: {
             bsonType: "object",
             description: "must be an object and is required"
           }
         }
       }
     }
   });
   ```
