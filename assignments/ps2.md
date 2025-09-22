# Problem Set 2: Modular Design

## 1) Concept Questions

1. **Contexts.** The *NonceGeneration* concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?

    - The contexts allow the existence of multiple used set of Strings, for which there may be duplicates between different contexts but not within the same context. For the URL shortening app, a context will end up being a shortURLBase (a domain name) because each base must have unique used short URL suffixes, but these can be repeated between bases and still result in different link addresses.


2. **Storing used strings.** Why must the *NonceGeneration* store sets of used strings? One simple way to implement the *NonceGeneration* is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an abstraction function.)

    - The *NonceGeneration* must store sets of used strings because every new string must be unique for a given context, so the state must remember what strings are already taken and thus no longer allowed. If a counter is maintained for each context, then given a counter value of c, the set of used strings would be {f(1), f(2), f(3), ..., f(c)}, with f being some function called upon the current counter value that takes in number and outputs a string (such as simply converting the number to a string).

    <!-- ,  each generated string could be the result of a function upon the current counter value, and the current counter value would be equal to the size of the set of used strings. -->

    <!-- In this case, the abstraction function from the counter to the set of used Strings is: AF(counter, context) = the size |counter| set of used strings for context "context". -->

3. **Words as nonces.** One option for nonce generation is to use common dictionary words (in the style of [yellkey.com](https://www.yellkey.com/), for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the *NonceGeneration* concept to realize this idea?

    - From the user's perspective, one advantage of this scheme is that the links are easier to copy down or type into a browser, but one disadvantage is that the link's don't give the user any indication of the web address that they're navigating to. To realize this idea, I'd change the *NoneGeneration* concept's state to also associate a dictionary with each context (along with an incrementing counter), and upon each call to generate, the counter value would be used to index into the dictionary and select a new, unique word each time.

## 2) Synchronization Questions

1. **Partial matching.** In the first sync (called *generate*), the
*Request.shortenUrl* action in the when clause includes the shortUrlBase argument but not the targetUrl argument. In the second sync (called *register*) both appear. Why is this?

    - For the generate sync, the user simply wants to generate a nonce. Thus, only the NonceGeneration concept is required, so the shortUrlBase arguement is sufficient for the generate action's context parameter. However, for the register sync, the user wants to create a specific link matching from a shortUrlSuffix to a targetUrl, so the URLShortening concept is needed. Its register action requires both the shortUrlBase and the targetUrl as parameters, so these two arguements must both appear in the second sync.

<!-- - generate just creates nonce, register actually matches/pairs it up? -->

  <!-- actions
    generate (context: Context) : (nonce: String)
      effect returns a nonce that is not already used by this context

  sync generate
  when Request.shortenUrl (shortUrlBase)
  then NonceGeneration.generate (context: shortUrlBase)

  sync register
  when
    Request.shortenUrl (targetUrl, shortUrlBase)
    NonceGeneration.generate (): (nonce)
  then UrlShortening.register (shortUrlSuffix: nonce, shortUrlBase, targetUrl) -->


2. **Omitting names.** The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn’t this convention used in every case?

    - This convention isn't used in every case because sometimes the name of the variable doesn't match up with, or is more specific than, the argument/result name, as a result of being carried over from a previous call. For instance, this is the case for the nonce variable name in the register sync and the shortURL variable name in the setExpiry sync.

3. **Inclusion of request.** Why is the request action included in the first two syncs but not the third one?

    - The request action is only included in the first two syncs because generating a nonce and registering a link are both actions that are directly initiatied by the user, so they happen as the result of a request. However, the setExpiry sync occurs automatically after a link is registered, so it doesn't require a request on behalf of the user.

    <!-- - Because expiration is only set when it's actually registered, which the user doesn't initiate, but is rather done once the nonce is generated, etc. -->

4. **Fixed domain.** Suppose the application did not support alternative domain names, and always used a fixed one such as “bit.ly.” How would you change the synchronizations to implement this?

    - To implement fixed domain names, the syncs would change such that there would be no shortUrlBase parameter in the when clause of the generate or register syncs, and neither NonceGeneration.generate nor UrlShortening.register would require a context argument. Instead, the short URL base could be hard-coded into the UrlShortening.register action to be used automatically without any user input.

    <!-- always pass in bit.ly? or don't pass in anything at all? wouldn't need sets of contexts in the state
    - request.shortenURL would no longer take in the base parameter; wouldn't need sets of contexts in the state because would only have 1 context (or could instead always pass in bit.ly no matter what arguement was passed in) -->

5. **Adding a sync.** These synchronizations are not complete; in particular, they don’t do anything when a resource expires. Write a sync for this case, using appropriate actions from the *ExpiringResource* and *URLShortening* concepts.

    **sync** expire\
    **when** ExpiringResource.expireResource () : (resource: shortUrl) \
    **then** URLShortening.delete (shortUrl)


## 3) Extending the Design

<!-- 1. Design a couple of additional concepts to realize this extension, and write them out in full (but including only the essential actions and state). It should not be necessary to make any changes to the existing concepts. -->

### 1. Concept Definitions

#### 1) Resource Counter

**concept** ResourceCounter [Resource]

**purpose** keep track of a resource's accesses

**principle** a resource is associated with a count that can then be repeatedly incremented, until the resource is deleted

**state**

&nbsp; a set of Resources with \
&nbsp;&nbsp;&nbsp; a count Number

**actions**

&nbsp; make (resource: Resource): (resource: Resource) \
&nbsp;&nbsp;&nbsp; **requires** resource is not already in set of resources \
&nbsp;&nbsp;&nbsp; **effects** associates resource with a count set to 0 and returns resource

&nbsp; increment (resource: Resource) : (resource: Resource) \
&nbsp;&nbsp;&nbsp; **requires** resource exists in set of resources \
&nbsp;&nbsp;&nbsp; **effects** increments resource's count by 1 and returns resource

<!-- &nbsp; authorize (creator: User, Counter) \
&nbsp;&nbsp;&nbsp; **requires** counter exists \ -->
<!-- &nbsp;&nbsp;&nbsp; **effects** creates and returns a new user with the given -->

&nbsp; delete (resource: Resource) \
&nbsp;&nbsp;&nbsp; **requires** resource exists in set of resources \
&nbsp;&nbsp;&nbsp; **effects** remove resource from set of resources

#### 2) Resource Hider

**concept** ResourceHider [Resource, User]

**purpose** manage access permissions among a resource's users

**principle** a resource is associated with a creator who is the only user that is authorized to access the resource, until the resource is deleted

**state**

&nbsp; a set of Resources with \
&nbsp;&nbsp;&nbsp; a creator User

**actions**

&nbsp; make (resource: Resource, creator: User): (resource: Resource) \
&nbsp;&nbsp;&nbsp; **requires** resource is not already in set of resources \
&nbsp;&nbsp;&nbsp; **effects** associates resource with creator and returns resource


&nbsp; authorize (resource: Resource, user: User) \
&nbsp;&nbsp;&nbsp; **requires** resource is in set of resources and user is its creator

&nbsp; delete (resource: Resource, user: User): (resource: Resource) \
&nbsp;&nbsp;&nbsp; **requires** resource exists in set of resources and user is its creator \
&nbsp;&nbsp;&nbsp; **effects** removes resource from set of resources

### 2. Syncs

<!--
2. Specify three essential synchronizations with your new concepts: one that happens when shortenings are created; one when shortenings are translated to targets; and one when a user examines analytics. -->

**sync** create\
**when** Request.shortenUrl(shortUrlBase, user) \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NonceGeneration.generate(context: shortUrlBase): (shortUrl) \
**then** ResourceCounter.make (resource: shortUrl) : (resource: shortUrl)

**sync** translate\
**when** URLShortening.register (shortUrlSuffix, shortUrlBase, targetUrl) : (shortUrl) \
**where** creator is the user associated with shortUrl in ResourceHider's state \
**then** ResourceHider.make (resource: shortUrl, creator) : (resource: shortUrl)

**sync** examine\
**when** Request.viewAnalytics (shortUrl, user) \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResourceHider.authorize (resource: shortUrl, user) \
**where** ResourceCounter's state associates shortUrl's shortening (from the UrlShortening's state) with count \
**then** Request.response (count)

### 3. Questions

As a way to assess the modularity of your solution, consider each of the following feature requests, to be included along with analytics. For each one, outline how it might be realized (eg, by changing or adding a concept or a sync), or argue that the feature would be undesirable and should not be included:
- Allowing users to choose their own short URLs;
    - To implement: Add an action to the NonceGeneration concept that takes in a user's chosen short string for a given context and returns the short string if it's unique.
- Using the “word as nonce” strategy to generate more memorable short URLs;
    - To implement: Update the NonceGeneration concept's state to hold a list of common dictionary words for each context that a counter associated with the context indexes into for each generate call in order to select a new, unique word each time.
- Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL;
    - To implement: Update the ResourceCounter concept's state to store target URL's instead of short URL's (and also update the create sync), and update the ResourceHider concept's state to store target URL's with a set of creators for each. Then, any time the same target URL is part of a translation, that user would be added to the link's creators in the ResourceHider concept; any incrementation of a short URL that's associated with the target URL in the URLShortening concept would increment the target URL's counter in the ResourceCounter concept; and any one of a link's creators would be given permission to view the analytics through ResourceHider's authorize method.
- Generate short URLs that are not easily guessed;
    - To implement: Change the NonceGeneration concept's generate action to create random sequences of characters that are then checked against the used set of strings before being returned.
- Supporting reporting of analytics to creators of short URLs who have not registered as user.
    - This feature is undesirable because it makes it more difficult to determine who should have permission to view short URL analytics. For instance, users would have to be defined through their IP address or a similar method, which would lead to potential issues such as that anyone using the same computer could also access the analytics, or that the user would lose their access priviledges if they attempted to view the analytics on another device.


<!-- # Questions
- why aren't there any arguements for the register sync second function call?? (do I leave out arguments if not important for sync?)
- sync question 2 - how to use just bit.ly -->
