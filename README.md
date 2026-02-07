# Wedding & Event Invitation System

## 1. Overview

A web-based application that allows event hosts to manage guest lists, send invitations, and collect RSVPs for weddings and special events.

## 2. User Roles

### 2.1 Event Host (Administrator)
- The person(s) organizing the event
- Full access to create/edit guest lists, send invitations, and view all responses
- Single user account per event (can share login credentials if multiple people need access)

### 2.2 Guest (Invitee)
- Recipients of event invitations
- Can view event details and submit RSVP for their family group
- No traditional login required - access via unique magic link sent to their email

## 3. Guest List Management

### 3.1 Family Structure
A **Family** is the basic organizational unit consisting of:
- **Family Name** (e.g., "The Smith Family")
- **Primary Contact**: One adult with full contact information
  - First Name
  - Last Name
  - Email Address (required)
  - Phone Number
  - Special Notes (optional text field)
- **Additional Family Members**: Other adults and children in the household
  - First Name
  - Last Name
  - Age Group (Adult/Child)
  - Email/Phone (optional)

**Rules:**
- Each family must have exactly one primary contact with a valid email
- The primary contact receives the invitation and can RSVP for the entire family
- Children under 18 are grouped with their parents
- Plus-ones can be added as additional family members or as separate single-person families
- Complex family situations (separated parents, etc.) are handled by creating separate family entries

### 3.2 Guest List Operations
Hosts can:
- Add new families manually
- Edit family information
- Delete families
- Import families via CSV upload (template provided)
- Export complete guest list to CSV
- Search/filter by family name, RSVP status, or notes

## 4. Event Information

Hosts configure basic event details:
- Event Name (e.g., "Sarah & John's Wedding")
- Event Date & Time
- Venue Name & Address
- Optional: Additional details/message (reception information, dress code, parking instructions, etc.)

## 5. Invitation System

### 5.1 Sending Invitations
- Hosts can send invitations to all families or selected families
- Invitations sent via email to the primary contact's email address
- Each invitation contains:
  - Event details
  - Unique RSVP link (magic link - no password required)
  - Personal greeting using family name

### 5.2 Invitation Status Tracking
Each family has a status:
- **Not Sent**: Invitation hasn't been sent yet
- **Sent**: Invitation sent, awaiting response
- **Responded**: RSVP completed

## 6. RSVP Process

### 6.1 Guest Access
- Guest clicks unique link in email invitation
- Taken directly to RSVP form (no login required)
- Form pre-populated with their family member names

### 6.2 RSVP Form
Guests indicate for each family member:
- Attending: Yes or No
- Meal Preference: (if enabled) Dropdown with host-defined options (e.g., Chicken, Beef, Vegetarian, Fish)
- Dietary Restrictions: Optional text field

Additional fields:
- General comments/notes (optional)
- Submit button

### 6.3 RSVP Rules
- Primary contact can RSVP for all family members at once
- Can mark some family members attending and others not attending
- Can edit RSVP by clicking the link again (until deadline, if set)
- Receives confirmation email after submitting

### 6.4 RSVP Deadline
- Optional deadline date configured by host
- After deadline, RSVP links are disabled (guests see "RSVP period has closed")
- Host can extend or remove deadline at any time

## 7. Host Dashboard

### 7.1 Overview Statistics
- Total families invited
- Total guests invited (individual count)
- Total guests attending
- Total guests not attending
- Number awaiting response

### 7.2 Guest List View
Tabular display showing:
- Family name
- Primary contact name
- Number in party
- Number attending
- RSVP status (Not Sent / Sent / Responded)
- Invitation sent date
- RSVP received date

### 7.3 Detailed RSVP View
Expandable view for each family showing:
- Each family member's name
- Their attendance status
- Meal preference (if collected)
- Dietary restrictions
- Guest comments

### 7.4 Meal Count Summary
If meal preferences enabled:
- Breakdown by meal type (e.g., 45 Chicken, 32 Beef, 18 Vegetarian)
- Total count

### 7.5 Reports & Export
- Export all RSVPs to CSV
- Export meal counts
- Printable guest list
- List of families who haven't responded (for follow-up)

## 8. Technical Requirements

### 8.1 Email Functionality
- Automated email sending for invitations
- Confirmation emails after RSVP submission
- Email templates with event branding
- Ability to send reminder emails to non-responders

### 8.2 Security
- Unique, unguessable tokens for guest RSVP links
- Host login protected by password
- HTTPS for all connections
- Links expire after event date + 30 days

### 8.3 Data Storage
- All guest information stored securely
- Host owns all data
- Ability to delete event and all associated data

### 8.4 Responsive Design
- Works on desktop, tablet, and mobile devices
- Mobile-friendly RSVP form for guests
- Optimized dashboard for hosts on all devices

## 9. Optional Features (Future Enhancements)

- SMS invitations as alternative to email
- Multiple events per account
- Photo gallery for event
- Registry links
- Thank you note tracking post-event
- Seating chart integration
- Guest messaging/questions

## 10. User Workflows

### 10.1 Host Workflow
1. Create account and event
2. Add event details (date, venue, etc.)
3. Build guest list (manually or import CSV)
4. Configure RSVP options (meal choices, deadline)
5. Send invitations
6. Monitor responses in dashboard
7. Send reminders to non-responders
8. Export final counts and meal preferences
9. View/print guest list for event day

### 10.2 Guest Workflow
1. Receive invitation email
2. Click RSVP link
3. View event details
4. Indicate attendance for each family member
5. Select meal preferences if applicable
6. Add any dietary restrictions or comments
7. Submit RSVP
8. Receive confirmation email

## 11. Techonology stack:
* Nextjs app hosted on Cloudflare workers
* Tailwindcss and Shadcn for the UI
* Cloudflare D1 for database
* resend.com for emailing out

## 12. Architecture of this repository
/app
  /(host)          # Protected routes for event hosts
    /dashboard
    /guests
    /invitations
  /(guest)         # Public routes for RSVP
    /rsvp/[token]
  /api
    /rsvp
    /invitations
    /families
