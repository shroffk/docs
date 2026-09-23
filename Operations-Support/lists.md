This document outlines the existing CAD lists and database tables that must be updated to support both injector infrastructure and operations as well as support for EIC construction activities.

# Lists
* [action please]( https://www.cadops.bnl.gov/AGS/Operations/ActPls2/)
* [call in](https://www.cadops.bnl.gov/Operations/personnel/call_in_lists/call_in_list.php?group=Controls)
* [diagnostics help](https://www.cadops.bnl.gov/cgi-loc/Controls.pl)

The action please system (Wenge) is a legacy CAD communication mechanism mainly used by Operations to request support for applications and systems (not just Controls)

The diagnostics help page is the custom URL for the Controls Software call-in list.  The one used by MCR is managed by the general web application (Leve)

# Current Status
Decisions are needed about how we expect to use these lists going forward.
1. Do we work on complete replacement tools?
2. Do we update the existing systems to meld both injector operations and eic construction tasks?
3. How do we identify experts and update supervisors where necessary?
4. Do the call-in lists require updates to the scheduling? Is off-hours support still required or is CAS involvement sufficient?
## Action Please
The action please system has dozens of entries that have either not been assigned or not addressed.  Many of the entries are considered "Low" priority.  Furthermore, some of the assignments that have already been made may need to be reassigned (Sam, Arthur) while others (Seth, John) should be re-visited to distribute to other Controls members.

There are several open entries for software that should no longer be necessary - AGS and RHIC applications.  The database should be pruned to remove any entries that are no longer required - A review of individual entries should be made to make this determination.

Additionally, the action please system includes supervisors for notification purposes.  The database that defines the supervisor list will need to be updated.

<b>Question</b> - Should this be the same system we use long-term or do we use a more modern solution?

## Software Call-In List
The Controls Software Diagnostics page includes both the call-in list and the contact people for systems, applications, and servers.  The call-in portion is synchronized with the MCR view of the list.

The call-in portion updates once per week on Monday and currently contains four pairs of software developers.  A separate list is used for system administration and contains a rotating schedule of the three system administrators.

<b>Question</b> - Do the members of these lists need to be udpated/changed?  How should we handle off-hours support?

Similar to the action please system, there are many applications listed that are no longer being used.  The list can probably be pruned a great deal but discussions should be had before making the decision to remove entries.

John is listed as primary contact on many applications.  After pruning, a decision needs to be made on how to re-assign these entries.  There are also many other entries where the responsible people are no longer employed at BNL.

# Database Tables
The database infrastructure is in the process of transitioning from Sybase to MySQL.  Several tables have staff information that requires attention.

This is a list of databases (and specific tables where relevant) where employee name information is stored
* ACTroubleLog
* GenInfo (CADEmployeeList)
* actionPlease
* callIn
* diagnostics (application, contactPersons)