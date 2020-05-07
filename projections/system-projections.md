---
outputFileName: index.html
---

<!-- TODO: retrofit to shopping cart examples? -->

# System projections

Event Store ships with four built in projections.

-   By Category (`$by_category`)
-   By Event Type (`$by_event_type`)
- By Correlation ID (`$by_correlation_id`)
-   Stream by Category (`$stream_by_category`)
-   Streams (`$streams`)

## Enabling system projections

When you start Event Store from a fresh database, these projections are present but disabled and querying their statuses returns `Stopped`. You can enable a projection by issuing a request which switches the status of the projection from `Stopped` to `Running`.

> QUESTION: So they cannot be started with command line options?

### [HTTP API](#tab/tabid-5)

```bash
curl -i -X POST "http://{event-store-ip}:{ext-http-port}/projection/{projection-name}/command/enable" -H "accept:application/json" -H "Content-Length:0" -u admin:changeit
```

### [.NET Client](#tab/tabid-6)

<!-- TODO: Is there a .NET equivelant? -->

* * *

## By category

The `$by_category` (_http://127.0.0.1:2113/projection/$by_category_) projection links existing events from streams to a new stream with a `$ce-` prefix (a category) by splitting a stream `id` by a configurable separator.

> QUESTION: Stream ID - is that the same as Stream Name, and if so why not stick to one term?

> COMMENT: It also creates new streams or? Please show / explain

> QUESTION: `ce_` - why? Acronym Category Events? 

```text
first
-
```
> COMMENT: This box above makes no sense? What does it mean? I can see it is the projection definition in the web interface, but ??

You can configure the separator, as well as where to split the stream `id`. You can edit the projection and provide your own values if the defaults don't fit your particular scenario.

The first parameter specifies how the separator is used, and the possible values for that parameter is `first` or `last`. The second parameter is the separator, and can be any character.

> QUESTION: - so the two items on the lines in the projection definition are parameters? It says `source` in the interface. 

For example, if the body of the projection is `first` and `-`, for a stream id of `account-1`, the stream name the projection creates is `$ce-account`.

If the body of the projection is `last` and `-`, for a stream id of `shopping-cart-1`, the stream name the projection creates is `$ce-shopping-cart`.

The use case of this project is subscribing to all events within a category.

> COMMENT: But, isn't there only a single $by_category projection? So you can only do that for one kind of streams, and not use last for some and first for others? Examples please. Elaborate.

## By event type

The `$by_event_type` (_http://127.0.0.1:2113/projection/$by_event_type_) projection links existing events from streams to a new stream with a stream id in the format `$et-{event-type}`.

> QUESTION: So this is like the $all projection, i.e. all events in the system, with the stream name added somewhere (since you don't bother to show)?

You cannot configure this projection.

## By correlation ID

The `$by_correlation_id` (_http://127.0.0.1:2113/projection/$by_correlation_id_) projection links existing events from projections to a new stream with a stream id in the format `$bc-<correlation id>`.

The projection takes one parameter, a JSON string as a projection source:

```json
{"correlationIdProperty":"$myCorrelationId"}
```
> DOH: WTF. What is a correlation ID - correlation to what? bc_ means what? Why would you want that? Existing events from projections - all projections in the system???


## Stream by category

The `$stream_by_category` (_http://127.0.0.1:2113/projection/$by_category_) projection links existing events from streams to a new stream with a `$category` prefix by splitting a stream `id` by a configurable separator.

```text
first
-
```

By default the `$stream_by_category` projection links existing events from a stream id with a name such as `account-1` to a stream called `$category-account`. You can configure the separator as well as where to split the stream `id`. You can edit the projection and provide your own values if the defaults don't fit your particular scenario.

The first parameter specifies how the separator is used, and the possible values for that parameter is `first` or `last`. The second parameter is the separator, and can be any character.

For example, if the body of the projection is `first` and `-`, for a stream id of `account-1`, the stream name the projection creates is `$category-account`, and the `account-1` stream is linked to it. Future streams prefixed with `account-` are likewise linked to the newly created `$category-account` stream.

If the body of the projection is last and `-`, for a stream id of `shopping-cart-1`, the stream name the projection creates is `$category-shopping-cart`, and the `shopping-cart-1` stream is linked to it.  Future streams whose left-side split by the __last__ '-' is `shopping-cart`, are likewise linked to the newly created `$category-shopping-cart` stream.

The use case of this projection is subscribing to all stream instances of a category.

> COMMENT: Help! I give up. 

## Streams

The `$streams` (_http://127.0.0.1:2113/projection/$streams_) projection links existing events from streams to a stream named `$streams`

You cannot configure this projection.

> Q?? So this links all events from all streams to a stream call streams? Eh - why? And different from $all?
