# ServiceNow Incident Management – Customization POC

## Project Overview

This project is a **ServiceNow Incident Management customization POC** that demonstrates different enhancements to the Incident Management process.

The project focuses on improving incident assignment, reassignment tracking, mandatory work notes, incident closure, user guidance, and data migration/configuration using ServiceNow platform features such as:

- Client Scripts
- Business Rules
- UI Policies
- Data Policies
- Dictionary Overrides
- Custom Fields
- Update Sets
- Transform Maps
- UI Actions

The configurations were implemented and tested on a custom Incident table (`POC_Incident_table`).

---

## Project Objectives

The main objectives of this project are to:

- Improve the Incident Assignment Group selection experience.
- Make work notes mandatory when an assignment group is selected.
- Provide help information for Impact and Urgency fields.
- Track the number of times an incident is reassigned.
- Automatically populate closure-related information.
- Make closure notes mandatory.
- Automatically capture the user who closes an incident.
- Demonstrate related Change Request functionality.
- Demonstrate Update Sets and Transform Maps for moving and importing configuration/data.

---

## Features Implemented

### 1. Assignment Group – Tree Picker

The Assignment Group lookup is configured to display groups in a **tree/hierarchical structure** instead of a standard list.

This makes it easier to navigate assignment groups when they have a parent-child hierarchy.

The configuration uses the `Tree picker` attribute along with Dictionary Overrides where required.

> **ServiceNow concepts:** Dictionary Entry, Dictionary Override, Reference Field, Tree Picker

---

### 2. Mandatory Work Notes for Assignment Group Changes

When an Incident is assigned to an Assignment Group, the **Work Notes** field is made mandatory.

This was demonstrated using:

- UI Policy
- Client Script
- Data Policy

The Client Script dynamically changes the mandatory state of the Work Notes field depending on whether an Assignment Group is selected.

Example logic:

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {

    if (isLoading || newValue == '') {
        return;
    }

    if (g_form.getValue('assignment_group')) {
        g_form.setMandatory('work_notes', true);
    } else {
        g_form.setMandatory('work_notes', false);
    }
}
```
### 3.Help Icons for Impact and Urgency 
**Help icons should be displayed next to the Impact and Urgency fields to provide guidance to users when selecting the appropriate values.**
```javascript
function onLoad() {
    g_form.addDecoration(
        'impact',
        'icon-help',
        'select correct impact'
    );
    g_form.addDecoration(
        'urgency',
        'icon-help',
        'select correct urgency'
    );
}
```
The icon-help decoration displays a help icon next to each field.
The result is that users can see the help icons while working with the Incident form.
### 4.Incident Reassignment Counter
The project tracks how many times an Incident's Assignment Group has changed.
A custom field is created to store the reassignment count.

#### Custom Field
Field Label: Reassignment count
Field Name: u_reassignment_count
Type: Integer
#### Business Rule

A Business Rule checks whether the Assignment Group has changed.

If it has changed, the reassignment counter is incremented.

```javascript (function executeRule(current, previous /*null when async*/) {

    if (current.assignment_group.changes()) {

        if (current.assignment_group.nil()) {
            return;
        }

        current.u_reassignment_count =
            (previous.u_reassignment_count || 0) + 1;
      }
    })(current, previous);
```

### 5.Display Reassignment Count

A Client Script is used to retrieve the reassignment count from the Incident record.

```javascript
function onLoad() {

    var reassignmentCount =
        g_form.getValue('u_reassignment_count');

    alert(
        "The reassignment count is " +
        reassignmentCount
    );

}
```
The script uses:

```g_form.getValue('u_reassignment_count')```

to retrieve the value stored in the custom field.

The value is then displayed to the user.

### 6. Auto-Populate Closure Information
The project demonstrates automatically populating information when an Incident reaches the closed state.

#### Client Script

A Client Script checks the Incident's state.

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {

    if (isLoading || newValue === '') {
        return;
    }

    var state = g_form.getValue('state');

    if (state == 7) {

        g_form.setValue(
            'description',
            'closed by ' + g_user.user_ID
        );

    } else {

        g_form.clearValue('description');

    }

}
```
The script:

Checks the Incident state.
Determines whether the Incident is closed.
Retrieves the current user's ID.
Populates the field with the closure information.

### 7.Custom Closure Fields

The Incident form was customized with additional closure-related fields.

The custom section contains fields such as:

Closed By
POC Closed Notes
Close notes

These fields are used to capture additional information when closing an Incident.

The project documentation shows these custom fields within the Incident form under the POC Closure Notes section.

### 8. Mandatory Closure Notes

Closure notes should be mandatory when completing an Incident.

#### Implementation

A UI Policy is configured to make the custom closure notes field mandatory.

UI Policy:
Make POC Closure Notes mandatory

Field:
POC Closed Notes

Mandatory:
True

This ensures that users provide closure information before completing the Incident.

### 9. Related Change Request

The project also demonstrates a Related Change Request functionality.

A UI Action is configured to perform an action related to the Change Request.

The implementation uses ServiceNow information such as:

```gs.getProperty('glide.servlet.uri')```

and the current record information.

The UI Action can be used to navigate from the current Incident context to the relevant Change Request functionality.

This demonstrates the use of UI Actions and server-side ServiceNow scripting.

### 10. Automatically Populate Closed By
When an Incident is closed, the Closed By field should contain the user who performed the closure.

#### Business Rule

A Business Rule checks whether the Incident state changes to closed.

```javascript
(function executeRule(current, previous /*null when async*/) {

    if (current.state.changesTo('closed')) {

        current.u_closed_by = gs.getUserID();

    }

})(current, previous);
```
#### Process
``` text
Incident
    ↓
State changes to Closed
    ↓
Business Rule executes
    ↓
gs.getUserID()
    ↓
Closed By field is populated
```
This removes the need for the user to manually enter who closed the Incident.

### 11. Update Sets
* The project demonstrates the use of Update Sets to capture ServiceNow configuration changes.
* Update Sets can be used to group configuration changes so that they can be moved between ServiceNow environments.
* The project contains Update Sets associated with the POC configuration.
* Examples shown in the project include:
```
POC_CHNG_Table
POC_INC_Table
POC_PRC_Table
POC_Service_catalog
```
The Update Set records show their application, state, creation information, and other configuration details.

### 12. Incident Management Table and Records

The project uses a custom Incident table:

POC_Incident_table

The Incident list contains records with fields such as:

* Number
* Opened
* Short description
* Caller
* Priority
* State
* Category

The project demonstrates multiple Incident records being created and displayed in the Incident list.

This provides the data used to test the different Incident Management customizations.

### 13. Import Sets and Transform Maps

The project demonstrates importing Incident-related data into ServiceNow and transforming the imported data.

#### Process

The overall process is:
```text
Source Data
     ↓
Import Set
     ↓
Transform Map
     ↓
Transform
     ↓
Incident Records
```
## Steps Demonstrated
#### Step 1 – Import Data\

Data is imported into ServiceNow using the import functionality.\

#### Step 2 – Configure Transform Map\

A Transform Map is used to map and transform the imported data.\

#### Step 3 – Run Transformation\

The transformation process is executed.\

#### Step 4 – Verify Transform History\

The Transform History is checked to verify the result.\

The Transform History displays information such as:\
```text
State
Completed
Run time
Import set
Total
Inserts
Updates
Ignored
Skipped
Errors
Transform Map
```
#### Step 5 – Verify Imported Records

After the transformation, the imported Incident records can be viewed in the Incident Import table.

The project documentation shows the imported Incident numbers after the transformation is completed.

### Overall Project Flow

The major customizations implemented in this project can be summarized as:
```text
                    ServiceNow
                        │
                        ▼
              Incident Management
                        │
        ┌───────────────┼────────────────┐
        │               │                │
        ▼               ▼                ▼
 Assignment Group    Incident Form    Data Import
        │               │                │
        ▼               │                ▼
   Tree Picker          │          Import Set
        │               │                │
        ▼               ▼                ▼
 Assignment Group   Client Scripts   Transform Map
 Changes            UI Policies          │
        │               │                ▼
        ▼               ▼          Imported Records
 Work Notes         Business Rules
 Mandatory              │
        │               ▼
        └───────► Closure Process
                       │
                       ▼
                Closed By User
                       │
                       ▼
                Closure Notes
```
