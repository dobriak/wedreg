# Spec: Family Management

## ADDED Requirements

### Requirement: Create family
The system SHALL allow hosts to create a new family with a family name, primary contact, and optional additional family members.

#### Scenario: Create family with only primary contact
- **WHEN** host submits family form with family name, primary contact first name, primary contact last name, and primary contact email
- **THEN** system creates a family record with exactly one family member (the primary contact)
- **AND** system marks that member as primary_contact = true
- **AND** system validates that email is in valid email format
- **AND** system returns the created family with generated ID

#### Scenario: Create family with additional members
- **WHEN** host submits family form with family name, primary contact, and one or more additional family members
- **THEN** system creates a family record with all specified members
- **AND** system marks only the primary contact as primary_contact = true
- **AND** system marks all other members as primary_contact = false
- **AND** system sets age_group for each member based on input (Adult/Child)

#### Scenario: Create family fails with invalid email
- **WHEN** host submits family form with primary contact email that is not a valid email format
- **THEN** system rejects the submission with validation error "Invalid email format"
- **AND** system does not create any family record

#### Scenario: Create family fails without primary contact email
- **WHEN** host submits family form without providing primary contact email
- **THEN** system rejects the submission with validation error "Primary contact email is required"
- **AND** system does not create any family record

#### Scenario: Create family fails without family name
- **WHEN** host submits family form without providing family name
- **THEN** system rejects the submission with validation error "Family name is required"
- **AND** system does not create any family record

### Requirement: Read family details
The system SHALL allow hosts to retrieve detailed information about a specific family including all family members.

#### Scenario: Retrieve existing family
- **WHEN** host requests family by valid ID
- **THEN** system returns family details including family name, creation date, update date
- **AND** system returns all family members with their names, age group, primary contact status, email, phone, and special notes
- **AND** system returns members sorted with primary contact first

#### Scenario: Retrieve non-existent family
- **WHEN** host requests family by invalid ID
- **THEN** system returns 404 Not Found error
- **AND** error message indicates family not found

### Requirement: Update family information
The system SHALL allow hosts to update family-level information (family name, notes) and individual member information.

#### Scenario: Update family name
- **WHEN** host submits update to family name for existing family
- **THEN** system updates the family name
- **AND** system updates the updated_at timestamp
- **AND** system returns updated family details

#### Scenario: Update primary contact information
- **WHEN** host submits update to primary contact's email, phone, or notes
- **THEN** system updates the specified fields for that member
- **AND** system validates email format if email is changed
- **AND** system requires email to remain present (cannot be set to null)

#### Scenario: Update additional member information
- **WHEN** host submits update to additional member's name, age group, email, phone, or notes
- **THEN** system updates the specified fields for that member
- **AND** system allows email and phone to remain optional for additional members

#### Scenario: Update fails on non-existent family
- **WHEN** host submits update to family ID that does not exist
- **THEN** system returns 404 Not Found error

### Requirement: Delete family
The system SHALL allow hosts to delete a family and all associated family members.

#### Scenario: Delete existing family
- **WHEN** host requests deletion of existing family by ID
- **THEN** system deletes all family members associated with that family
- **AND** system deletes the family record
- **AND** system returns 204 No Content on success

#### Scenario: Delete fails on non-existent family
- **WHEN** host requests deletion of non-existent family ID
- **THEN** system returns 404 Not Found error

### Requirement: Add family member to existing family
The system SHALL allow hosts to add new family members to an existing family.

#### Scenario: Add adult member to family
- **WHEN** host submits new family member with first name, last name, and age_group = Adult
- **THEN** system creates new family member record linked to family
- **AND** system sets primary_contact = false
- **AND** system returns updated family with all members

#### Scenario: Add child member to family
- **WHEN** host submits new family member with first name, last name, and age_group = Child
- **THEN** system creates new family member record linked to family
- **AND** system sets primary_contact = false
- **AND** system allows email and phone to be optional (null)

#### Scenario: Add member fails on non-existent family
- **WHEN** host submits new family member to family ID that does not exist
- **THEN** system returns 404 Not Found error

### Requirement: Update family member
The system SHALL allow hosts to update individual family member details.

#### Scenario: Update member name
- **WHEN** host submits update to family member's first or last name
- **THEN** system updates the name field
- **AND** system returns updated member details

#### Scenario: Update member age group
- **WHEN** host submits update to family member's age_group
- **THEN** system updates the age_group field
- **AND** system validates value is either "Adult" or "Child"

#### Scenario: Update primary contact email
- **WHEN** host submits update to primary member's email
- **THEN** system validates email format
- **AND** system updates email if valid
- **AND** system rejects update if email is invalid

### Requirement: Delete family member
The System SHALL allow hosts to delete individual family members, with validation to prevent deleting the last member.

#### Scenario: Delete additional family member
- **WHEN** host requests deletion of non-primary family member
- **THEN** system deletes the family member
- **AND** system returns 204 No Content

#### Scenario: Delete primary contact when other members exist
- **WHEN** host requests deletion of primary contact when family has other members
- **THEN** system rejects deletion with error "Cannot delete primary contact. Delete the family or assign a new primary contact first."
- **AND** system does not delete any members

#### Scenario: Delete last remaining member (family deletion)
- **WHEN** host requests deletion of the only member in a family
- **THEN** system deletes the family member
- **AND** system deletes the family record (empty family cannot exist)

### Requirement: List all families
The system SHALL allow hosts to retrieve a paginated list of all families.

#### Scenario: List families with default pagination
- **WHEN** host requests families list without specifying page or limit
- **THEN** system returns first 50 families
- **AND** system includes total count of families
- **AND** system includes pagination metadata (page, limit, total_pages)

#### Scenario: List families with custom pagination
- **WHEN** host requests families list with page=2 and limit=25
- **THEN** system returns families 26-50
- **AND** system respects the specified limit
- **AND** system includes pagination metadata

#### Scenario: List families sorted by family name
- **WHEN** host requests families list with sort=family_name
- **THEN** system returns families sorted alphabetically by family name

### Requirement: Validate family structure constraints
The system SHALL enforce validation rules for family structure.

#### Scenario: Enforce exactly one primary contact per family
- **WHEN** system creates or updates a family
- **THEN** system ensures exactly one member has primary_contact = true
- **AND** system prevents setting primary_contact = true on multiple members
- **AND** system prevents setting primary_contact = false on the only member

#### Scenario: Enforce required fields for primary contact
- **WHEN** system validates primary contact member
- **THEN** system requires first_name, last_name, and email to be present
- **AND** system validates email format
- **AND** system allows phone and notes to be optional

#### Scenario: Allow optional fields for additional members
- **WHEN** system validates additional family members
- **THEN** system requires first_name and last_name to be present
- **AND** system allows email and phone to be optional (null)
- **AND** system allows notes to be optional
