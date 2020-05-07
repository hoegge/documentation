---
outputFileName: index.html
---

# User defined projections

<!-- TODO: Again refactor to shopping cart? -->

You write user defined projections in JavaScript. For example, the `my_demo_projection_result` projection below counts the number of `myEventType` events from the `account-1` stream. It then uses the `transformBy` function to change the final state:

```JavaScript
options({
	resultStreamName: "my_demo_projection_result",
	$includeLinks: false,
	reorderEvents: false,
	processingLag: 0
})

fromStream('account-1')
.when({
	$init:function(){
		return {
			count: 0
		}
	},
	myEventType: function(state, event){
		state.count += 1;
	}
})
.transformBy(function(state){
	state.count = 10;
})
.outputState()
```

<!-- TODO: Show example output, see above comment -->

## User defined projections API

### Options

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Description</th>
            <th>Notes</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>`resultStreamName`</code></td>
            <td>Overrides the default resulting stream name for the `outputState()` transformation, which is `$projections-{projection-name}-result`.</td>
            <td>
            </td>
        </tr>
        <tr>
            <td><code>`$includeLinks`</code></td>
            <td>Configures the projection to include/exclude link to events.</td>
            <td>
                <b>Default: </b>`false`
            </td>
        </tr>
         <tr>
            <td><code>`processingLag`</code></td>
            <td>When `reorderEvents` is enabled, this value is used to compare the total milliseconds between the first and last events in the buffer and if the value is equal or greater, the events in the buffer are processed. The buffer is an ordered list of events.
            </td>
			<td>
                <b>Default: </b>`500ms`
                <p>
	                Only valid for `fromStreams()` selector
                </p>
            </td>
        </tr>
        <tr>
            <td><code>`reorderEvents`</code></td>
            <td>Process events by storing a buffer of events ordered by their prepare position</td>
			<td>
                <b>Default: </b>`false`
                <p>
	                Only valid for `fromStreams()` selector
                </p>
            </td>
        </tr>
	</tbody>
</table>

> QUESTION: So why is it called resultStreamName instead of outputStateName (since it only applies when using that, right?) **/hoegge**

> COMMENT: `$includeLinks`: What is that ? What links? Explain? **/hoegge**

> COMMENT: If you relate to reorderEvents, then describe that first. "The buffer is an ordered list of events" belongs to the description of reorderEvents, doesn't it? **/hoegge**

> QUESTION: What does reorderEvents do? You don't explain it - what it is used for and why. What is "prepare position"? **/hoegge**

## Selectors

The first function call in a query is a selector, that defines what input to process in the query.

<table>
    <thead>
        <tr>
            <th>Selector</th>
            <th>Description</th>
            <th>Notes</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>`fromAll()`</code></td>
            <td>Selects events from the `$all` stream.</td>
            <td>
            	<b>Provides</b>
            	<ul>
	            	<li>`partitionBy`</li>
	            	<li>`when`</li>
	            	<li>`foreachStream`</li>
	            	<li>`outputState`</li>
            	</ul>
            </td>
        </tr>
        <tr>
            <td><code>`fromCategory({category})`</code></td>
            <td>Selects events from the `$ce-{category}` stream.</td>
            <td>
            	<b>Provides</b>
            	<ul>
	            	<li>`partitionBy`</li>
	            	<li>`when`</li>
	            	<li>`foreachStream`</li>
	            	<li>`outputState`</li>
            	</ul>
            </td>
        </tr>
        <tr>
            <td><code>`fromStream({streamId})`</code></td>
            <td>Selects events from the {streamId} stream.</td>
            <td>
            	<b>Provides</b>
            	<ul>
	            	<li>`partitionBy`</li>
	            	<li>`when`</li>
	            	<li>`outputState`</li>
            	</ul>
            </td>
        </tr>
        <tr>
            <td><code>`fromStreams([]streams)`</code></td>
            <td>Selects events from the streams supplied.</td>
            <td>
            	<b>Provides</b>
            	<ul>
	            	<li>`partitionBy`</li>
	            	<li>`when`</li>
	            	<li>`outputState`</li>
            	</ul>
            </td>
        </tr>
        <tr>
            <td><code>`fromStreamsMatching(function filter)`</code></td>
            <td>Selects events from the `$all` stream that returns true for the given filter.</td>
            <td>
            	<b>Provides</b>
            	<ul>
	            	<li>`when`</li>
            	</ul>
            </td>
        </tr>
	</tbody>
</table>

> fromAll(): i.e. all events in the system? Are they always ordered correctly by storage order and then same order across streams always? What does "Provides mean? Should it say: Available filters / transformations?**/hoegge**

> fromCategory: so this is the same as writing fromStream("$ce-category")? Or does it result in multiple streams (which "provides" indicates)?**/hoegge**

> fromStreams([]streams): streams[] ? So they result in a single interlaces stream? Any indication of what events come from what stream? Not multiple streams as above, since it does not support foreachStream? Use case please?**/hoegge**

> fromStreamsMatching(functionfilter): So this function can select events by - what? What should the functionfilter (I assume that is a function) receive as parameters? Is it matching streams or matching events. This is simply the same as undocumented.**/hoegge**

## Filters/Transformations

The streams selected by the selector can subsequently be processed by compatible filters / transformations. The selectors table show what filters are available for each selector. Filters can be chained by subsequent compatible filters (listed in right column in table below).

> COMMENT: Header is filters/transformations. The table column is Filter/Partition. What do you mean and what is the difference if any? What is a partition (in your ES world)?

<table>
    <thead>
        <tr>
            <th>Filter/Partition</th>
            <th>Description</th>
            <th>Notes</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>`when(handlers)`</code></td>
            <td>Allows only the given events of a particular to pass through the projection.</td>
            <td>
            	<b>Provides</b>
            	<ul>
	            	<li>`$defines_state_transform`</li>
	            	<li>`transformBy`</li>
	            	<li>`filterBy`</li>
	            	<li>`outputTo`</li>
	            	<li>`outputState`</li>
            	</ul>
            </td>
        </tr>
        <tr>
            <td><code>`foreachStream()`</code></td>
            <td>Partitions the state for each of the streams provided.</td>
            <td>
            	<b>Provides</b>
            	<ul>
	            	<li>`when`</li>
            	</ul>
            </td>
        </tr>
        <tr>
            <td><code>`outputState()`</code></td>
            <td>If the projection maintains state, setting this option produces a stream called `$projections-{projection-name}-result` with the state as the event body.</td>
            <td>
            	<b>Provides</b>
            	<ul>
	            	<li>`transformBy`</li>
	            	<li>`filterBy`</li>
	            	<li>`outputTo`</li>
            	</ul>
            </td>
        </tr>
        <tr>
            <td><code>`partitionBy(function(event))`</code></td>
            <td>Partitions a projection by the partition returned from the handler.</td>
            <td>
            	<b>Provides</b>
            	<ul>
	            	<li>`when`</li>
            	</ul>
            </td>
        </tr>
        <tr>
            <td><code>`transformBy(function(state))`</code></td>
            <td>Provides the ability to transform the state of a projection by the provided handler.</td>
            <td>
            	<b>Provides</b>
            	<ul>
	            	<li>`transformBy`</li>
	            	<li>`filterBy`</li>
	            	<li>`outputState`</li>
	            	<li>`outputTo`</li>
            	</ul>
            </td>
        </tr>
        <tr>
            <td><code>`filterBy(function(state))`</code></td>
            <td>Causes projection results to be `null` for any `state` that returns a `false` value from the given predicate.</td>
            <td>
            	<b>Provides</b>
            	<ul>
	            	<li>`transformBy`</li>
	            	<li>`filterBy`</li>
	            	<li>`outputState`</li>
	            	<li>`outputTo`</li>
            	</ul>
            </td>
        </tr>
	</tbody>
</table>

> when(): Allows only the given events of a particular ___what___ to pass through the projection? "handlers" - what is that and how defined? From looking at the docs forever, I assume it is the functions in the supplied JS object, with the eventtype as keys, but again - this is not described at all. **/hoegge**

> foreachStream(): Generates an independent state instance for each stream provided? But from the selectors description none of them provide multiple streams. I GUESS fromStreams does, but does some of the others? Hard to know since it is not documented. **/hoegge**

> outputState(): Does it produce only that then? Or will it also emit events processed by the query? Is that what is determined by the emit setting in the query? You should explain the difference between state-generating querys and stream-generating - and if possible - that you can do both. **/hoegge**

> partitionBy(function(event)): Yeah - whatever that means. Come on - explain what it is. What is a partition? What does it mean to partition a projection? Can you partition a stream? It is all quite obscure. **/hoegge**

> transformBy: again - incromprehensible? Transform state - don't you do that for every event? Or is it a kind of "finally" transformation? How do you transform events? (which is important for versioning) Can't see that described anywhere

> filterBy(function(state)): So this filters state (eradicates it). Any way to do the same for events?

## Handlers

Each handler is provided with the current state of the projection as well as the event that triggered the handler (except for the $init handler? **/hoegge**). The event provided through the handler contains the following properties.

-   `isJson`: true/false
-   `data`: {}
-   `body`: s{}
-   `bodyRaw`: string
-   `sequenceNumber`: integer
-   `metadataRaw`: {}
-   `linkMetadataRaw`: string
-   `partition`: string
- `eventType`: string

> QUESTION: What is `body`? What does the s in s{} mean?
> Q: What is sequenceNumber? The event number (as you call it in the web-interface instead of sequenceNumber) (in the stream)? **/hoegge**

> Q: In a projection / query - is the sequence number the number in the projection / query or the original event number of the event? In the webUI there is something called "Name" number@streamName, which seems to be the orginal number. Where does that come from? **/hoegge**

> Q: What is linkMetadataRaw?**/hoegge**

> Q: Again you use partition without having ever defined what it means.**/hoegge**


<table>
    <thead>
        <tr>
            <th>Handler</th>
            <th>Description</th>
            <th>Notes</th>
        </tr>
    </thead>
    <tbody>
         <tr>
            <td><code>{event-type}</code></td>
            <td>
                When using `fromAll()` and 2 or more event type handlers are specified and the `$by_event_type` projection is enabled and running, the projection starts as a `fromStreams($et-event-type-foo, $et-event-type-bar)` until the projection has caught up and moves to reading from the transaction log (i.e. from `$all`).
            </td>
            <td>
            </td>
        </tr>
        <tr>
            <td><code>`$init`</code></td>
            <td>Provide the initialization for a projection.</td>
            <td>
            	Commonly used to setup the initial state for a projection.
            </td>
        </tr>
        <tr>
            <td><code>`$initShared`</code></td>
            <td>Provide the initialization for a projection where the projection is possibly partitioned.</td>
            <td>
            </td>
        </tr>
        <tr>
            <td><code>`$any`</code></td>
            <td>Event type pattern match that match any event type.</td>
            <td>
            	Commonly used when the user is interested in any event type from the selector.
            </td>
        </tr>
        <tr>
            <td><code>`$deleted`</code></td>
            <td>Called upon the deletion of a stream.</td>
            <td>
                Can only be used with `foreachStream`
            </td>
        </tr>
	</tbody>
</table>

> event-type description: Incomprehensible - too short and only useful if you already know. But finally and indication of why you would run system projections, even though it is not explained at all. **/hoegge**

> $init: "Commonly used"??? Isn't it only used for that? And isn't it also what the description says? Any parameters available here? **/hoegge**

> $initShared: what, why? Explain.  **/hoegge**

> $any: Notes: no sh.. Sherl... Maybe should be the first handler to decribe?

> $deleted: Called upon? So this is called when a stream is deleted only? BTW, what happens when you delete a stream? Not described is it? Seems like the last event number is kept so if you use that stream again it continues from the last number? (and you get blank pages in the web UI since it cannot figure that out)  **/hoegge**



## Functions

> QUESTION: Where and how are these used? If inside handlers, then write that. Describe the parameters, what are they? What is event in linkTo - and ID or data?

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Description</th>
            <th>Notes</th>
        </tr>
    </thead>
    <tbody>
         <tr>
            <td><code>`emit(streamId, eventType, eventBody, metadata)`</code></td>
            <td>Writes an event to the designated stream</td>
            <td>
            </td>
        </tr>
        <tr>
            <td><code>`linkTo(streamId, event, metadata)`</code></td>
            <td>Writes a link to event to the designated stream</td>
            <td>
            </td>
        </tr>
    </tbody>
</table>
