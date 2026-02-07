# Spec: Guest List Search

## ADDED Requirements

### Requirement: Search families by family name
The system SHALL allow hosts to search for families by family name with case-insensitive matching.

#### Scenario: Search with partial family name match
- **WHEN** host enters "Smith" in search box
- **THEN** system returns all families with "Smith" anywhere in family name (e.g., "The Smith Family", "Smith-Jones", "The Johnson-Smith Family")
- **AND** search is case-insensitive (matches "smith", "SMITH", "Smith")
- **AND** results are sorted alphabetically by family name

#### Scenario: Search with no results
- **WHEN** host enters family name that does not match any families
- **THEN** system returns empty result set
- **AND** system displays message "No families found matching your search"

#### Scenario: Clear search
- **WHEN** host clears search box or clicks "Clear" button
- **THEN** system resets view to show all families
- **AND** search box is emptied

#### Scenario: Search with empty query
- **WHEN** host leaves search box empty and triggers search
- **THEN** system shows all families
- **AND** system behaves as if no search filter is applied

### Requirement: Search families by primary contact name
The system SHALL allow hosts to search for families by primary contact's first or last name.

#### Scenario: Search by primary contact first name
- **WHEN** host enters "John" in search box
- **THEN** system returns all families where primary contact's first name contains "John"
- **AND** search is case-insensitive

#### Scenario: Search by primary contact last name
- **WHEN** host enters "Williams" in search box
- **THEN** system returns all families where primary contact's last name contains "Williams"
- **AND** search is case-insensitive

#### Scenario: Search with full name
- **WHEN** host enters "John Williams" in search box
- **THEN** system returns families where primary contact's first name AND last name match (in any order)
- **AND** system returns "John Williams", "Williams John", and partial matches like "Johnny Williams"

### Requirement: Search families by email address
The system SHALL allow hosts to search for families by primary contact's email address.

#### Scenario: Search by full email
- **WHEN** host enters "john.smith@example.com" in search box
- **THEN** system returns family with exact email match
- **AND** search is case-insensitive for local part and domain

#### Scenario: Search by partial email
- **WHEN** host enters "@example.com" in search box
- **THEN** system returns all families with email addresses in example.com domain
- **AND** search matches email suffix

#### Scenario: Search by email local part
- **WHEN** host enters "john.smith" in search box
- **THEN** system returns families with email addresses containing "john.smith" before the @ symbol

### Requirement: Search families by family member name
The system SHALL allow hosts to search for families by any family member's name (not just primary contact).

#### Scenario: Search by additional member name
- **WHEN** host enters "Sarah" in search box
- **THEN** system returns families where ANY family member's first or last name contains "Sarah"
- **AND** this includes primary contact and additional members
- **AND** results indicate which family member matched (highlighted in UI)

#### Scenario: Search with multiple matching members
- **WHEN** host enters "John" and family has two members named "John"
- **THEN** system returns the family once
- **AND** system indicates multiple matches within the family

### Requirement: Filter families by RSVP status
The system SHALL allow hosts to filter the guest list by invitation or RSVP status.

#### Scenario: Filter by "Not Sent" status
- **WHEN** host selects "Not Sent" filter
- **THEN** system shows only families where invitation has not been sent
- **AND** system displays filter badge "Status: Not Sent"

#### Scenario: Filter by "Sent" status
- **WHEN** host selects "Sent" filter
- **THEN** system shows only families where invitation has been sent but no RSVP received
- **AND** system displays filter badge "Status: Sent (Awaiting Response)"

#### Scenario: Filter by "Responded" status
- **WHEN** host selects "Responded" filter
- **THEN** system shows only families where RSVP has been submitted
- **AND** system displays filter badge "Status: Responded"

#### Scenario: Clear status filter
- **WHEN** host clicks "Clear Filter" or selects "All" status
- **THEN** system removes status filter
- **AND** system shows all families regardless of invitation/RSVP status

### Requirement: Filter families by party size
The system SHALL allow hosts to filter families by the number of people in the party.

#### Scenario: Filter by party size range
- **WHEN** host selects party size filter "5-10 people"
- **THEN** system shows only families with 5-10 total members
- **AND** system displays filter badge "Party Size: 5-10"

#### Scenario: Filter by single person
- **WHEN** host selects party size filter "1 person"
- **THEN** system shows only families with exactly 1 member
- **AND** system includes single-person families created for plus-ones

#### Scenario: Filter by large families
- **WHEN** host selects party size filter "10+ people"
- **THEN** system shows only families with 10 or more members

### Requirement: Combine multiple search and filter criteria
The system SHALL allow hosts to combine search queries with multiple filters.

#### Scenario: Combine search with status filter
- **WHEN** host enters search query "Smith" AND selects status filter "Responded"
- **THEN** system shows only families matching "Smith" AND with RSVP status "Responded"
- **AND** system displays both search query and filter badge

#### Scenario: Combine search with party size filter
- **WHEN** host enters search query "Williams" AND selects party size filter "5-10 people"
- **THEN** system shows only families matching "Williams" AND with 5-10 members

#### Scenario: Combine status and party size filters
- **WHEN** host selects status filter "Not Sent" AND party size filter "1 person"
- **THEN** system shows only uninvited single-person families

#### Scenario: Remove one criterion while keeping others
- **WHEN** host has combined search and filters AND clears search box
- **THEN** system maintains active filters
- **AND** system refreshes results based on remaining filters

### Requirement: Real-time search results
The system SHALL provide real-time search results as user types (debounced).

#### Scenario: Type-ahead search updates results
- **WHEN** host types characters in search box
- **THEN** system waits 300ms after user stops typing
- **AND** system updates results with matching families
- **AND** user can continue typing while search executes

#### Scenario: Show loading state during search
- **WHEN** system is executing search query
- **THEN** system displays loading indicator in search results area
- **AND** existing results remain visible until new results load

### Requirement: Server-side search for large lists
The system SHALL use server-side search when guest list exceeds 500 families.

#### Scenario: Switch to server-side search automatically
- **WHEN** guest list has 501 families
- **THEN** system uses server-side search endpoint
- **AND** search query is sent to API at `/api/families/search?q=<query>`
- **AND** API returns paginated results

#### Scenario: Client-side search for small lists
- **WHEN** guest list has 300 families
- **THEN** system uses client-side search
- **AND** search is performed on cached family data in browser
- **AND** no API call is made

#### Scenario: Manual override for search mode
- **WHEN** guest list has 200 families but user prefers server-side search
- **THEN** system provides optional toggle "Use server search"
- **AND** user can enable server-side search manually

### Requirement: Search performance and limits
The system SHALL provide performant search with reasonable limits.

#### Scenario: Search completes within acceptable time
- **WHEN** host executes search on guest list with 1000 families
- **THEN** search completes and displays results within 2 seconds

#### Scenario: Limit search results
- **WHEN** search query matches more than 100 families
- **THEN** system displays first 100 results
- **AND** system shows message "Showing 100 of 245 results. Refine your search."
- **AND** system does not paginate search results (user must refine query)

#### Scenario: Debounce rapid search queries
- **WHEN** host types multiple characters quickly (e.g., "Smith" in 1 second)
- **THEN** system only executes search for final "Smith" query
- **AND** intermediate queries are cancelled

### Requirement: Preserve search state across navigation
The system SHALL preserve search and filter state when navigating to other pages and returning.

#### Scenario: Navigate away and return
- **WHEN** host has active search/filter AND navigates to dashboard
- **AND** host clicks "Guests" link to return
- **THEN** system restores previous search query and filters
- **AND** results display matches what user saw before leaving

#### Scenario: Share search URL
- **WHEN** host has active search and filters
- **THEN** URL query parameters reflect search state (e.g., `/guests?q=smith&status=sent&size=5-10`)
- **AND** user can bookmark or share URL to share filtered view

#### Scenario: Clear search from URL
- **WHEN** user visits clean URL without query parameters
- **THEN** system displays all families without filters
- **AND** search box is empty
