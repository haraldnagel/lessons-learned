# Introduction
This document represents some lessons learned from developing software in an environment with a team spread across the country working in an older waterfall/spiral environment on line-of-business CRUD database applications. Not all of it is relevant in an agile environment or on a smaller team. It is presented as a list for consideration and/or as talking points to start conversations.

# Key principles

- Just becuase you *can* do a thing doesn't mean you *should* do a thing
- "Just In Case" (or "Justin Case" as we call him) is not a paying client - we don't implement things that we *might* need later
- For each technology selected, question "is there an older, more proven, less exciting technology which could be selected to fit here?" ([source](https://blog.pinboard.in/2010/01/technical_underpinnings/))

# Application settings

- Select reasonable defaults if a setting is not present
  - All settings possible should be OK if their value is null/undefined
  - Is that setting *really* required? Can we pick a sensible default in case the user doesn’t manually configure it?
- On application load/start check all configuration file paths and ensure required ones exist
- Describe settings as close to the place that they are configured as possible (e.g. comments in the config file)

# Databases

- Select a uniform naming scheme and follow it, for example:
  - Development database: `<product name>_dev` (or similar)
  - Test database: `<product_name>_test` (or similar)
  - Production database: `<product_name>`
- Create and follow a database update process to ensure that updates are not clobbered
- Put as much of the database stuff in revision control as possible (e.g. DDL scripts, code models, or a code-first approach)
- Queries should probably have an `ORDER BY` clause, there are few business lists that shouldn’t be ordered by default
- Data should be imported for test in ways which cause the IDs to be non-sequential in order to root out problems where things are sorted by ID or natural insert order
- All data queries which expect a specific number of results (e.g. 0, 1, 2) should verify that the correct number of results are returned before using them

# Development discipline

- If developers can't be trusted to check their assigned issues, they should be sent automatically via email (possibly daily)
- Comprehensive code reviews
  - Developer to take notes during review from reviewer
  - Developer distributes notes to code review team
  - Developer fixes their own problems
  - Re-review on any issues using notes as a source
  - It's not a personal attack, we learn by constructive criticism
- Before undertaking system construction, a review team should sign off on design and direction
- For all change requests, record who made the request and when/where/how
- All user-supplied documents must be logged including the context under which they were delivered, known issues with the document, date received, who sent them, who received them. These files should be stored somewhere where they can be kept perpetually and the log must be updated any time new files are received
- Project controls (management of project tasks, budget, and schedule) require time and budget, put them on the schedule and estimate
- All estimates should include documentation regarding the the approach used (for example, considering the situation where one developer creates the estimate but another is tagged to do the implementation)
- Time must be allocated for updating existing documentation to bring it current
- Test plans must be created by someone with a deep understanding of the requirements - ideally someone who has been involved with the project through the whole development process
- Never trust user input

# Email

- Email should be sent from a system/role address and not an individual user’s address
- Use a `Reply-To` header if replies should be directed to a monitored account
- If there is no specified `Reply-To` header and the source address is not monitored, notate clearly in the email that it is a "do not reply" situation
- Configure and verify SPF records
- Consider implementation of DKIM and DMARC (helpful resource: <https://dmarcian.com>)
- Always include the configuration ability to reroute all email from the system to a specific address for testing or find a service which will accept and log email (e.g. <https://mailtrap.io>)

# Implementation

- Everything should have an upper limit, even if it’s "ridiculous"
- Use constants for all hardcoded strings
- Document all magic numbers
- Code that is "no longer used" erodes efficiency in all areas: maintenance, testing, documentation. Remove it (it will be saved in revision control).

# Logging

- All applications should have a unified logging interface - all notifications/bugs/exceptions should be logged in the same place
- Logs must be reviewed during development and testing to ensure exceptions are resolved - are situations which regularly write exceptions at level `ERROR` to the log really exceptional?
- Logging should have levels and all messages should be logged with the appropriate level, this can be defined in project notes if there are multiple developers or it's not obvious what each level represents
- Which levels are written to the log file or database or are emailed to developers should be configurable at runtime
- Logs should be cleared prior to each run of the test plan so that errors which are logged can be reviewed and addressed
- Compiler directives (e.g. `#ifdef`) should be avoided for any error logging to ensure that developers see errors as end-users do
- Scheduled processes (i.e. run by Windows Task Manager or `cron`) need clear logging and time scheduled and budgeted for reviewing the logs (put this time in the schedule and budget)
- If logs are being written to a database
  - Consider using a separate database so that if the log database becomes full it does not affect the application database
  - A plan is necessary for truncating old logs or verifying available disk space and appropriate database growth settings
- Administrators may need the ability to view the log from within the application

# Maintenance programming

- Changes/enhancements should be made in as limited a way as possible to ensure there is minimal chance of introducing new bugs or widening the test window substantially
- Maintenance programming should follow the coding standards of the existing project even if they are not the current coding standards
- Resist the desire to refactor existing code which is working properly
- Always beware of second-system syndrome ("I'm going to redo this and do it right this time")

# Presentations

- Back-up presenter should have the application up in case there are any issues with the primary presenter
- Check display resolutions prior to the presentation, shoot for least common denominator if it's a shared-screen situation
- Avoid having multiple people presenting if possible (avoid hand-overs and switching of who is presenting)
- Do not use transitions for network shared-screen presentations

# Production/deployment

- Log all databases and deployment sites with why they were created and update whenever data is modified
  - This log should be kept up-to-date with any relevant events
  - Could be a text file, spreadsheet, SharePoint list, etc.
  - Not for client eyes, can be casual
  - Chronological or reverse chronological order with what happened when - pick one and stick to it
  - Must be religiously updated or isn’t useful at all
- Search all deployable files for internal server names prior to deployment or distribution to the client

# Revision control

- **Always**, even on quick hacks. The initial investment is negligible with Git
- The only way to validate that everything is properly in revision control is to clear out the working directory, do a full clone from revision control, and perform a build
  - Don't remove your working copy for this, just compress it - that way if you've forgotten to check in critical files you still have tyhem
- Processor power is cheap, consider setting up a clean virtual machine so that you can verify the dependencies specified are truly the only dependencies needed to build
- Branches should include some sort indication (perhaps a document in the root of the branch) describing who made it, why it was made, and its expected lifetime (i.e. think about running across the branch in a year - will you know what it's there for or if it's safe to delete?)
- Establish and follow best practices for including documentation in the source tree (e.g. GitHub/Visual Studio Online standard of a root level `README.md`)

# System dependencies

- Always include a check which can be run independent of the application (during deployment) for:
  - File system access and permissions
  - Database access
  - Sending a test email
  - Any external APIs, especially if they are accessed over the network
- If the application requires the database to function, connectivity should be verified before any application operations take place so that database connectivity errors are presented in a clear manner and not mistaken for application errors
- When possible, allow the administrator to see credentials being used in the software (minus passwords)

# Web sites

- Consider whether uploaded files should be stored apart from the hosted Web site files and not made accessible without passing through code (i.e. don't dump uploaded files in the main Web share path)
