# Spec: Guest List I/O

## ADDED Requirements

### Requirement: Export guest list to CSV
The system SHALL allow hosts to export the complete guest list to a CSV file containing family and family member data.

#### Scenario: Export all families
- **WHEN** host clicks "Export Guest List" button
- **THEN** system downloads a CSV file named `guest-list-YYYY-MM-DD.csv`
- **AND** CSV contains one row per family with columns: Family Name, Primary Contact, Primary Email, Primary Phone, Party Size, Notes, Created Date, Updated Date
- **AND** CSV includes all families regardless of invitation status
- **AND** CSV rows are sorted alphabetically by Family Name
- **AND** CSV uses standard RFC 4180 format (comma-delimited, quoted fields, UTF-8 encoding)

#### Scenario: Export with family members included
- **WHEN** host requests export with include_members=true
- **THEN** CSV includes additional columns for each family member (up to 10 members): Member 1 Name, Member 1 Age Group, Member 1 Email, Member 1 Phone, Member 1 Notes, etc.
- **AND** family members are displayed in the order they appear in the family

#### Scenario: Export empty guest list
- **WHEN** host clicks "Export Guest List" button when no families exist
- **THEN** system downloads a CSV file with only headers (no data rows)

#### Scenario: Export handles special characters
- **WHEN** family names or member names contain commas, quotes, or newlines
- **THEN** CSV properly escapes these characters according to RFC 4180
- **AND** values containing commas are wrapped in quotes
- **AND** quotes within values are escaped with double quotes

### Requirement: Import families from CSV
The system SHALL allow hosts to import families from a CSV file, creating families and family members from the data.

#### Scenario: Import valid CSV with multiple families
- **WHEN** host uploads a CSV file with valid family data
- **THEN** system validates the CSV format and required columns
- **AND** system creates families for each valid row
- **AND** system creates family members for each row
- **AND** system generates unique IDs for each family and member
- **AND** system sets created_at and updated_at timestamps
- **AND** system returns summary with total rows processed, families created, and any errors

#### Scenario: Import CSV with validation errors
- **WHEN** host uploads a CSV file with some invalid rows
- **THEN** system processes all valid rows
- **AND** system skips invalid rows with clear error messages including row number
- **AND** system returns summary showing total rows, successful imports, and failed imports
- **AND** system provides detailed error list for each failed row (e.g., "Row 5: Invalid email format", "Row 7: Missing family name")

#### Scenario: Import CSV fails with invalid format
- **WHEN** host uploads a CSV file that cannot be parsed (malformed, wrong encoding, etc.)
- **THEN** system rejects the entire import
- **AND** system returns error "Unable to parse CSV file. Please ensure it is a valid CSV."
- **AND** system does not create any families

#### Scenario: Import CSV with missing required columns
- **WHEN** host uploads a CSV without required columns (Family Name, Primary Contact First Name, Primary Contact Last Name, Primary Contact Email)
- **THEN** system rejects the import
- **AND** system returns error "CSV missing required columns. Please use the provided template."

#### Scenario: Import with duplicate family names
- **WHEN** host imports a CSV with family names that already exist in the database
- **THEN** system creates duplicate families with same family name
- **AND** system does not attempt to merge or update existing families
- **AND** system warns user about potential duplicates in import summary

#### Scenario: Import handles large CSV files
- **WHEN** host uploads a CSV with more than 100 families
- **THEN** system processes the file in batches
- **AND** system provides progress feedback to user
- **AND** system completes import without timeout
- **AND** system uses transaction to ensure atomicity (all or nothing for each batch)

### Requirement: Provide CSV import template
The system SHALL provide a downloadable CSV template with proper structure and example data.

#### Scenario: Download template
- **WHEN** host clicks "Download Template" button
- **THEN** system downloads a CSV file named `guest-list-template.csv`
- **AND** template includes header row with all supported columns
- **AND** template includes 3 example rows demonstrating valid data
- **AND** template includes a comment row (starting with #) explaining required vs optional columns

#### Scenario: Template includes required columns
- **WHEN** host opens the template file
- **THEN** CSV includes required columns: Family Name, Primary Contact First Name, Primary Contact Last Name, Primary Contact Email
- **AND** CSV includes optional columns: Primary Contact Phone, Primary Contact Notes, Additional Members (JSON format)

#### Scenario: Template demonstrates JSON format for additional members
- **WHEN** host examines example rows in template
- **THEN** template shows Additional Members column with JSON array format: `[{"first_name":"Jane","last_name":"Smith","age_group":"Adult","email":"jane@example.com","phone":"555-0101","notes":""}]`
- **AND** template includes examples of adults and children
- **AND** template shows empty array [] for families with no additional members

### Requirement: Validate CSV data during import
The system SHALL validate all CSV data before creating families and members.

#### Scenario: Validate email format
- **WHEN** CSV row contains email that is not a valid email format
- **THEN** system marks row as invalid
- **AND** error message indicates "Invalid email format for primary contact"
- **AND** system does not create family from that row

#### Scenario: Validate age group values
- **WHEN** CSV row contains age_group that is not "Adult" or "Child"
- **THEN** system marks row as invalid
- **AND** error message indicates "Age group must be 'Adult' or 'Child'"

#### Scenario: Validate JSON format for additional members
- **WHEN** CSV row contains Additional Members column that is not valid JSON
- **THEN** system marks row as invalid
- **AND** error message indicates "Invalid JSON format for Additional Members"

#### Scenario: Validate family name is not empty
- **WHEN** CSV row has empty string for Family Name
- **THEN** system marks row as invalid
- **AND** error message indicates "Family name cannot be empty"

#### Scenario: Validate primary contact name fields
- **WHEN** CSV row has empty strings for Primary Contact First Name or Primary Contact Last Name
- **THEN** system marks row as invalid
- **AND** error message indicates "Primary contact name cannot be empty"

### Requirement: Provide import confirmation before processing
The system SHALL allow hosts to preview import data and confirm before creating families.

#### Scenario: Show import preview
- **WHEN** host uploads a CSV file
- **THEN** system parses and validates the file
- **AND** system displays preview showing: total rows, valid rows, invalid rows
- **AND** system shows sample of first 5 valid rows with data
- **AND** system shows list of error rows with error messages
- **AND** system presents "Confirm Import" and "Cancel" buttons

#### Scenario: Cancel import from preview
- **WHEN** host clicks "Cancel" button on import preview
- **THEN** system discards the uploaded file
- **AND** system does not create any families
- **AND** system returns to guest list view

#### Scenario: Confirm import from preview
- **WHEN** host clicks "Confirm Import" button on import preview
- **THEN** system creates all valid families
- **AND** system skips invalid rows
- **AND** system shows progress during import
- **AND** system displays final summary after completion
- **AND** system redirects to updated guest list

### Requirement: Handle CSV character encoding
The system SHALL properly handle different character encodings for CSV import and export.

#### Scenario: Import UTF-8 encoded CSV
- **WHEN** host uploads UTF-8 encoded CSV file
- **THEN** system correctly reads all characters including accented letters, emojis, and Unicode characters

#### Scenario: Import Latin-1 encoded CSV
- **WHEN** host uploads CSV file with Latin-1 encoding
- **THEN** system detects encoding or uses UTF-8 as default
- **AND** system reads characters correctly
- **AND** system handles encoding errors gracefully by replacing unsupported characters with question marks

#### Scenario: Export CSV with special characters
- **WHEN** family data contains accented characters or emojis
- **THEN** exported CSV file uses UTF-8 encoding with BOM for better Excel compatibility
- **AND** special characters are correctly preserved
