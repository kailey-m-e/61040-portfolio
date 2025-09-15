# Problem Set: Concept Design

## 1) Reading a Concept

1. **Invariants.** What are two invariants of the state? (*Hint*: one is about aggregation/counts of items, and one relates requests and purchases). Say which one is more important and why; identify the action whose design is most affected by it, and say how it preserves it.

    1. The count of an item in the registry is never negative.
    2. Every purchase from the registry is of a requested item.

    - The first invariant is more important because the count of an item in the registry represents how many of that gift the recipient would like to receive, which can never logically be less than zero.

    - The action whose design is most affected by the first invariant is purchase, which preserves the invariant using the pre-condition that the registry must have a request for the given item that's no less than the count of the item being purchased. In this way, the current count of the item minus the number of the item purchased will never be negative.


2. **Fixing an action.** Can you identify an action that potentially breaks this important invariant, and say how this might happen? How might this problem be fixed?

    - One action that potentially breaks the first invariant is addItem. For instance, if the count parameter is a negative number and the item doesn't already exist in the registry, then the item will be added to the registry with a negative count. This problem could be fixed by requiring that count is always a non-negative number.

3. **Inferring behavior.** The operational principle describes the typical scenario in which the registry is opened and eventually closed. But a concept specification often allows other scenarios. By reading the specs of the concept actions, say whether a registry can be opened and closed repeatedly. What is a reason to allow this?

    - Yes, a registry can be opened and closed repeatedly, because a registry always exists once it has been created. Thus, after it has been closed, the open action only requires that it exists and is inactive, which is true, so it can be opened again. Furthermore, after it has been opened, the close action only requires that it exists and is active, which is true, so it can be closed again; and so on.

    - One reason to allow opening and closing the registry repeatedly is so that a recipient can reopen a closed registry if a purchaser misses the registry's initial deadline but then still wants to buy a late gift, and then the recipient can close the registry again afterwards.

4. **Registry deletion.** There is no action to delete a registry. Would this matter in practice?

    - No, this likely wouldn't matter in practice because the current actions simulate the deletion of a registry: if a registry is closed and then never opened again, it will be permanently closed, which essentially provides the equivalent function to a registry being deleted.

5. **Queries.** What are two common queries likely to be executed against the concept state? (*Hint*: one is executed by a registry owner, and one by a giver of a gift.)

    - One common query that the registry owner will likely ask is to obtain the list of gifts that were purchased, and one common query that a giver will likely ask is to obtain the list of currently available items in the registry.

5. **Hiding purchases.** A common feature of gift registries is to allow the recipient to choose not to see purchases so that an element of surprise is retained. How would you augment the concept specification to support this?

    - To support the capability of hiding purchases, the registry could have a purchasesVisible flag. This flag would be set to either true or false via an input flag parameter from the recipient when the registry is made with the create action, and it would prevent the recipient from querying the state about what gifts were purchased.

6. **Generic types.** The User and Item types are specified as generic parameters. The Item type might be populated by SKU codes, for example. Explain why this is preferable to representing items with their names, descriptions, prices, etc.

    - This is preferable because by defining Users and Items in other concepts, the specific details of these types, which are not important to the fundamental functions of a GiftRegistration, are abstracted away. Thus, this allows Users and Items to be independently modified and used across a variety of other concepts while GiftRegistration, which is focused solely on defining a gift registry, remains unchanged.


## 2) Extending a Familiar Concept

### Concept Definition

**concept** PasswordAuthentication

**purpose** limit access to known users


**principle** after a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user

**state**

&nbsp; a set of Users with \
&nbsp;&nbsp;&nbsp; a username String \
&nbsp;&nbsp;&nbsp; a password String \
&nbsp;&nbsp;&nbsp; a token String \
&nbsp;&nbsp;&nbsp; a confirmed Flag


<!-- a set of Tokens with \
a User \
a token String -->

<!-- token only aassociated with one user -->

**actions**

&nbsp; register (username: String, password: String): (user: User, token: String) \
&nbsp;&nbsp;&nbsp; **requires** username doesn't exist among current users \
&nbsp;&nbsp;&nbsp; **effects** creates and returns a new user with the given username, password, and a False confirmed flag,  \
&nbsp;&nbsp;&nbsp; and returns a secret token associated with that user

&nbsp; confirm (username: String, token: String): (user: User) \
&nbsp;&nbsp;&nbsp; **requires** username matches a user whose secret token matches the given token and who hasn't been confirmed \
&nbsp;&nbsp;&nbsp; **effects** updates user's confirmed flag to True

&nbsp; authenticate (username: String, password: String): (user: User) \
&nbsp;&nbsp;&nbsp; **requires** username matches a user whose password matches the given password and who has been confirmed


<!-- **state**

a set of Users with
- a username String
- a password String

a set of Tokens with
- a user User
- a token String

a set of Statuses with
- a user User
- a registered Flag
- an authenticated Flag

**actions**

register (username: String, password: String): (user: User, token: String)

- **requires** username doesn't exist among current users

- **effects** creates and returns a new user with the given username and password that's marked as unregistered and unauthenticated, and returns a secret token associated with that user

confirm (username: String, token: String): (user: User)

- **requires** username matches a user that's unregistered and unauthenticated, and token matches that user's token

- **effects** mark user as registered

authenticate (username: String, password: String): (user: User)

- **requires** username matches a user that's  registered but unauthenticated, and password matches that user's password

- **effects** mark user as authenticated -->

### Questions

3. What essential invariant must hold on the state? How is it preserved?

    - The state must maintain the invariant that no two users have the same username. This is preserved because the requires clause of the register action means that a new user can only register with a given username if that username is unique among all users, and there is no action that modifies a username after it has been created.

## 3) Comparing Concepts

### Concept Definition

**concept** PersonalAccessToken [User, Scope]

**purpose** easily share personal permissions between GitHub users

**principle** after a user generates a personal access token, another user can use that token to authenticate for (certain scopes of) the token creator's permissions on GitHub until the token expires or is deleted

**state**

&nbsp; a set of Tokens with \
&nbsp;&nbsp;&nbsp; a creator User \
&nbsp;&nbsp;&nbsp; a name String \
&nbsp;&nbsp;&nbsp; a token String \
&nbsp;&nbsp;&nbsp; an expiration Date \
&nbsp;&nbsp;&nbsp; a set of Scopes

**actions**

&nbsp; generate(creator: User, name: String, expiration: Date, scopes: set of Scopes): (token: Token) \
&nbsp;&nbsp;&nbsp; **requires** short doesn't already exist among links \
&nbsp;&nbsp;&nbsp; **effects** creates and returns new token with creator, name, expiration data, set of scopes, and a uniquely generated token sequence

&nbsp; authenticate(token: Token, scope: Scope) \
&nbsp;&nbsp;&nbsp; **requires** scope is associated with token and token's expiration date falls after the current date

&nbsp; delete(user: User, token: Token) \
&nbsp;&nbsp;&nbsp; **requires** token exists and user is token's creator \
&nbsp;&nbsp;&nbsp; **effects** removes token from set of tokens

### Questions

- PasswordAuthentication and PersonalAccessToken differ because PasswordAuthentication requires that a user has a single password, which authorizes only the user for their personal capabilities. However, for the PersonalAccessToken concept, a user can create multiple tokens with different scopes each that are given to any number of other users, who then use one of the tokens to authenticate for the creator's personal permissions within scope. Thus, passwords and tokens differ in the number that a single user can create, and in which users can gain authorization with that password/token sequence.

- I think that the GitHub documentation could be improved by providing a clear table that compares and contrasts the details of passwords versus personal access tokens, as well as an example flow of how personal access tokens would be used. Finally, it would be more straightforward to explain how the tokens work before explaining the minute differences between the two types.

<!-- https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens -->

## 4) Defining Familiar Concepts

### 1. URL Shortener

**concept** URLShortener [User]

<!-- do I need person attached to each? add user, can edit later -->

**purpose** make long links easier to handle

**principle** a user manually or auto-generates a shortened suffix for a link, after which the new short link directs users to the same original destination web address as the original link, until it's deleted

**state**

&nbsp; a set of Links with \
&nbsp;&nbsp;&nbsp; a User \
&nbsp;&nbsp;&nbsp; a suffix String \
&nbsp;&nbsp;&nbsp; a desintation String

**actions**

<!-- autoShorten(originalLink: String): (shortLink: String)
- **requires**
- **effects** creates and returns a unique, shortened version of originalLink that's associated with originalLink

userShorten(originalLink: String, shortLink: String): (shortLink: String)
- **requires** shortenedSuffix doesn't exist among links
- **effects** associates the shortenedSuffix with the originalSuffix and returns the shortenedSuffix

deleteLink(shortenedSuffix: String)
- **requires** shortenedSuffix exists among links
- **effects** remove link with shortenedSuffix from links -->

&nbsp; autoShorten(user: User, destination: String): (link: Link) \
&nbsp;&nbsp;&nbsp; **effects** creates and returns new link with the user; a unique, automatically generated short suffix; and original destination

&nbsp; manualShorten(user: User, destination: String, suffix: String): (link: link) \
&nbsp;&nbsp;&nbsp; **requires** suffix doesn't already exist among links \
&nbsp;&nbsp;&nbsp; **effects** creates and returns new link with user, given suffix, and original destination

&nbsp; delete(user: User, link: Link) \
&nbsp;&nbsp;&nbsp; **requires** link is one of user's existing links \
&nbsp;&nbsp;&nbsp; **effects** removes link from set of links

<!-- (look at gift registry for example of checking and setting) -->
Notes: Links are deleted and remade by a creator rather than edited, and deletion by the creator is also the only means for a link to "expire."

### 2. Billable Hours Tracking

**concept** HourTracker [Employee, Project]

**purpose** simplify hourly billing of employees

**principle** an employee starts a work session by choosing a project and describing their work, and then closes the session when done

**state**
<!-- &nbsp; a set of Logs with \
&nbsp;&nbsp;&nbsp; an Employee \
&nbsp;&nbsp;&nbsp; a set of Sessions -->

&nbsp; a set of Sessions with \
&nbsp;&nbsp;&nbsp; an Employee \
&nbsp;&nbsp;&nbsp; a start DateTime \
&nbsp;&nbsp;&nbsp; an end DateTime \
&nbsp;&nbsp;&nbsp; a Project \
&nbsp;&nbsp;&nbsp; a description String

**actions**

&nbsp; start(employee: Employee, project: Project, description: String): (session: Session) \
&nbsp;&nbsp;&nbsp; **requires** there are no currently open (started and not ended) sessions for employee  \
&nbsp;&nbsp;&nbsp; **effects** creates and returns session with employee, project, and description; a start time set to the current date/time; and no end time

&nbsp; end(employee: Employee, session: Session) \
&nbsp;&nbsp;&nbsp; **requires** session was started, but not already ended, by employee \
&nbsp;&nbsp;&nbsp; **effects** sets session's end time to current date/time

&nbsp; editStart(employee: Employee, session: Session, start: DateTime): (session: Session) \
&nbsp;&nbsp;&nbsp; **requires** session was created by employee, and start time is not in the future and falls before session's end time (if one exists) \
&nbsp;&nbsp;&nbsp; **effects** updates session's start time and returns session

&nbsp; editEnd(employee: Employee, session: Session, end: DateTime): (session: Session) \
&nbsp;&nbsp;&nbsp; **requires** session was created by employee, and end time is not in the future and falls after session's start time (if one exists) \
&nbsp;&nbsp;&nbsp; **effects** updates session's start time and returns session

&nbsp; editProject(employee: Employee, session: Session, project: Project): (session: Session) \
&nbsp;&nbsp;&nbsp; **requires** session was created by employee \
&nbsp;&nbsp;&nbsp; **effects** updates session's project and returns session

&nbsp; editDescription(employee: Employee, session: Session, description: String): (session: Session) \
&nbsp;&nbsp;&nbsp; **requires** session was created by employee \
&nbsp;&nbsp;&nbsp; **effects** updates session's description and returns session

Notes: To handle the case of a user forgetting to end a session, that user can later end the session in their log using the editEnd action.

### 3. Conference Room Booking

<!-- https://tig.csail.mit.edu/events-reservations/reserve/ -->

**concept** RoomBooking [User]

**purpose** guarantee conference rooms are kept available for particular groups as needed

**principle** a user reserves a particular conference room for a particular time block, and then has the assurance that the room will be available for their group during that time, unless the booking is deleted

**state**

&nbsp; a set of Bookings with \
&nbsp;&nbsp;&nbsp; a name String \
&nbsp;&nbsp;&nbsp; a creator User \
&nbsp;&nbsp;&nbsp; an owners set of Users \
&nbsp;&nbsp;&nbsp; a Date \
&nbsp;&nbsp;&nbsp; a start Time \
&nbsp;&nbsp;&nbsp; a duration Number \
&nbsp;&nbsp;&nbsp; a room String \
&nbsp;&nbsp;&nbsp; a notes String

**actions**

&nbsp; make(name: String, creator: User, owners: set of Users, date: Date, start: Time, duration: Number, room: String, notes: String): (b: Booking) \
&nbsp;&nbsp;&nbsp; **requires** there is no booking for room during the time period of (start + duration) on date \
&nbsp;&nbsp;&nbsp; **effects** makes and returns a new booking with the creator and the given name, owners, date, start time, duration, room, and notes

&nbsp; editName(owner: User, b: Booking, name: String): (b: Booking) \
&nbsp;&nbsp;&nbsp; **requires** b is an existing booking and owner is one of that booking's owners \
&nbsp;&nbsp;&nbsp; **effects** updates b's name and returns b

&nbsp; editOwners(owner: User, b: Booking, owners: set of Users): (b: Booking) \
&nbsp;&nbsp;&nbsp; **requires** b is an existing booking and owner is one of that booking's owners \
&nbsp;&nbsp;&nbsp; **effects** updates b's owners and returns b

&nbsp; editTime(owner: User, b: Booking, start: Time): (b: Booking) \
&nbsp;&nbsp;&nbsp; **requires** b is an existing booking, owner is one of that booking's owners, and there is no other booking for room during the time period of (start + duration) on date **effects** updates b's start time and returns b

&nbsp; editDate(owner: User, b: Booking, date: Date): (b: Booking) \
&nbsp;&nbsp;&nbsp; **requires** b is an existing booking, owner is one of that booking's owners, and there is no other booking for room during the time period of (start + duration) on date **effects** updates b's date and returns b

&nbsp; editDuration(owner: User, b: Booking, duration: Number): (b: Booking) \
&nbsp;&nbsp;&nbsp; **requires** b is an existing booking, owner is one of that booking's owners, and there is no other booking for room during the time period of (start + duration) on date \
&nbsp;&nbsp;&nbsp; **effects** updates b's duration and returns b

&nbsp; editNotes(owner: User, b: Booking, notes: String): (b: Booking) \
&nbsp;&nbsp;&nbsp; **requires** b is an existing booking and owner is one of that booking's owners \
&nbsp;&nbsp;&nbsp; **effects** updates b's notes and returns b

&nbsp; delete(owner: User, b: Booking) \
&nbsp;&nbsp;&nbsp; **requires** b is an existing booking and owner is one of that booking's owners \
&nbsp;&nbsp;&nbsp; **effects** removes b from set of bookings
