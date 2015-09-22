Qi stream behaviors are applied to streams to effects how certain data read operations will be performed. A stream behavior must first be defined and then can be applied to a stream when it is created (`GetOrCreateStream` method) or updated (`UpdateStream` method). A stream behavior is always referenced with its `Id` property.

A stream behavior can be changed between reads to change how the read acts. A stream can also change the stream behavior it references at any time.

The default behavior for a stream (when a defined stream behavior is not applied to the stream) is `Mode = ‘Continuous’` and `ExtrapolationMode = ‘All’`. 

## Qi Stream Behavior Object

```
QiStreamExtrapolation ExtrapolationMode
string Id
QiStreamMode Mode
string Name
IList<QiStreamBehaviorOverride> Overrides 
```

- `Id` -- unique identifier used to reference this behavior
- `Name` -- Optional descriptor
- `Mode` -- behavior setting to be applied to all ‘value’ properties of events in the stream to which this is applied
- `ExtrapolationMode` -- controls extrapolation behavior on the ends of a range
- `Overrides` -- A list of overrides to specific properties
 

`QiStreamMode` is an enumeration whose permissible values are:
- Continuous (=0)
- StepwiseContinuousLeading (=1)
- StepwiseContinuousTrailing (=2)
- Discrete (=3)

`ExtrapolationMode` is an enumeration:
- All (=0)
- None (=1)
- Forward (=2)
- Backward (=3)


The `QiStreamBehaviorOverride` object has the following structure:
```
string QiTypePropertyId
QiStreamMode Mode
```

It is used to apply a behavior override to a specific property or properties of an event, rather than all properties of the event.


## Stream Behavior Mode
When running a query method, if an index lands between 2 values in the stream, then the stream behavior is used to determine what is returned. The mode for a stream behavior or override can be set to one of these values: 

- Continuous (default): value is interpolated using previous and next events (see chart below for exceptions)
- StepwiseContinuousLeading: value is obtained from previous event
- StepwiseContinuousTrailing: value is obtained from next event
- Discrete: NULL value is returned

There are cases where a null value cannot be used. For example when a `GetValue` call is made on a stream that has a behavior using a `Continuous` value for `Mode` and a property of the event type has a Discrete override, the call will attempt to set this Discrete property to ‘null’.  But in cases where this cannot be done (i.e. a non-nullable type) the default value will be used. 

The chart below describes how the types act when the mode is set to Continuous:

| Type	| When mode is Continuous and an index between events is addressed |
| ------ | ---------------------------------------------------------------- |
| Numeric Floating Point Types Single, Double, Decimal | Interpolation |
| Numeric Integer Types Int16, int32, int64, uint16, uint32, uint64, byte, Sbyte, Char | Interpolation (rounding) |
| Time related Types DateTime, DateTimeOffset, TimeSpan	| Interpolation |
| Nullable Types NullableBoolean, NullableChar, NullableSByte, NullableByte, NullableInt16, NullableUInt16, NullableInt32, NullableUInt32, NullableInt64, NullableUInt64, NullableSingle, NullableDouble, NullableDecimal, NullableDateTime, NullableGuid, NullableDateTimeOffset, NullableTimeSpan | Returns null (as if the mode were Discrete) |
| Array and List Types BooleanArray, CharArray, SByteArray,ByteArray, Int16Array, UInt16Array, Int32Array, UInt32Array, Int64Array, UInt64Array, SingleArray,DoubleArray, DecimalArray, DateTimeArray, StringArray, GuidArray, DateTimeOffsetArray, TimeSpanArray, VersionArray, IList | Returns null (as if the mode were Discrete) |
| String | Returns null (as if the mode were Discrete) |
| Boolean | Returns the value of nearest event |
| Enumeration Types SByteEnum, ByteEnum, Int16Enum, UInt16Enum, Int32Enum, UInt32Enum, Int64Enum,UInt64Enum | Returns ‘0’ which may be the value of a defined enumeration element. |
| Guid | Returns Guid.Empty   |
| QiType, QiTypeProperty | Returns null (as if the mode were Discrete) |
| Version | Returns null (as if the mode were Discrete) |
| IDictionary, IEnumerable | Returns null (as if the mode were Discrete) |

The behavior of all values in the stream event type will be controlled by the stream behavior `Mode` property. `Continuous` is the default if a defined stream behavior is not set. Individual event properties can be overridden to act as another behavior by setting the `Overrides` property. In this way the user can have different extrapolation behavior for different properties within the same event.  Note that when doing this, the main Behavior Mode is still used to determine whether an event is returned for an index between data. If the main behavior `Mode` property is set to ‘Discrete’, no event is returned for the call requiring a calculated value, regardless of any overrides.

##Stream Behavior ExtrapolationMode
- `All` (default):  extrapolation done at both start and end of data in stream. 
- `Forward`: extrapolation done at the end of the stream (not at the beginning). 
- `Backward`: extrapolation done at the beginning of the stream (not at the end). 
- `None`: no extrapolation done  

The `ExtrapolationMode` property applies to a stream in the following conditions:
- `GetValue` and `GetValues` when an index is used that is before or after all events in the stream
- `GetWindowValues` when the start index is before all events in the stream or when the end index is after all events in the stream
- `GetRangeValues` when the start index is before or after all events in the stream
- `GetIntervals` when indices are on each side of an interval

| Behavior | Extrapolation | Before Start of Stream | After End of Stream | Empty Stream |
| -------- | ------------- | ---------------------- | ------------------- | ------------ |
| Continuous | All | First Event Fields | Last Event Fields | Null |
| Continuous | None | Null | Null | Null |
| Continuous | Backward | First Event Fields | Null | Null |
| Continuous | Forward | Null | Last Event Fields | Null |
| Discrete | All | Null | Null | Null |
| Discrete | None | Null | Null | Null |
| Discrete | Backward | Null | Null | Null |
| Discrete | Forward | Null | Null | Null |
| StepwiseContinuousLeading | All | Null | Last Event Fields | Null |
| StepwiseContinuousLeading | None | Null | Null | Null |
| StepwiseContinuousLeading | Backward | Null | Null | Null |
| StepwiseContinuousLeading | Forward | Null | Last Event Fields | Null |
| StepwiseContinuousTrailing | All | First Event Fields | Null | Null |
| StepwiseContinuousTrailing | None | Null | Null | Null |
| StepwiseContinuousTrailing | Backward | First Event Fields | Null | Null |
| StepwiseContinuousTrailing | Forward | Null | Null | Null |

## Naming Rules for Behavior Identifiers
1.	Case sensitive
2.	Allows spaces.
   
## Qi Stream Behavior Methods

*DeleteBehavior*
```
void DeleteBehavior(string behaviorId);
Task DeleteBehaviorAsync(string behaviorId);
```

*REST*
```
Qi/Behaviors/{behaviorId}
```

HTTP DELETE

*Parameters*

`behaviorId` -- id of the behavior to delete; the behavior must not be associated with any streams

Deletes behavior from server.

*GetBehavior*
```
QiStreamBehavior GetBehavior(string behaviorId);
Task<QiStreamBehavior> GetBehaviorAsync(string behaviorId);
```
*REST*
```
Qi/Behaviors/{behaviorId}
```

HTTP GET

*Parameters*

`behaviorId` -- id of the behavior definition to retrieve

Gets a QiStreamBehavior object from server.

*GetBehaviors*
```
IEnumerable<QiStreamBehavior> GetBehaviors();
Task<IEnumerable<QiStreamBehavior>> GetBehaviorsAsync();
```

Returns IEnumerable of all behaviors for the tenant.

*GetOrCreateBehavior*
```
QiStreamBehavior GetOrCreateBehavior(QiStreamBehavior entity);
Task<QiStreamBehavior> GetOrCreateBehaviorAsync(QiStreamBehavior entity);
```

*REST*
```
Qi/Behaviors
```

HTTP POST
Body is serialized `QiStreamBehavior` entity

*Parameters*

`entity` -- a QiStreamBehavior object to add to the Qi Service for the current tenant.  
Creates a QiStreamBehavior (or returns it if it already exists). 

If `entity` already exists on the server by `Id`, that existing behavior is returned to the caller unchanged.

*UpdateBehavior*
```
void UpdateBehavior(string behaviorId, QiStreamBehavior entity);
Task UpdateBehaviorAsync(string behaviorId, QiStreamBehavior entity);
```
*REST*
```
Qi/Behaviors/{behaviorId}
```

HTTP PUT
Body is a serialization of the updated `QiStreamBehavior`

*Parameters*

`behaviorId` -- identifier of the stream behavior to update
`entity` -- updated stream behavior 

Permitted changes: 

- Override list
- BehaviorMode
- ExtrapolationMode
- Name

An override list can be added to add, remove or change the `Mode` on properties within the type. 
UpdateBehavior replaces the stream’s existing behavior with `entity`.  If certain aspects of the existing behavior are meant to remain, they must be included in entity.

This is a list of properties from the event type (`QiTypePropertyId`) that are to be given different behaviors (`Mode`). The overrides list is used in cases where the user desires the stream to have different behaviors for different values in a stream's events. 

`Extrapolation`
QiStreamExtrapolation can have one of 4 values.

1.	All
2.	Nonei
3.	Forward
4.	Backward

This indicates whether indices that are read before or after all data should attempt to return an event or not.