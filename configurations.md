# Configurations
Running a signal program depends on configurations defined outside of the signal progam, as described below.

Configurations are organized in a hierachy from general to specific.

Settings can be overriden by more specific configurations. For example, a setting defined in the regional
configurations can be overriden in a controller or intersection configuration.

```mermaid
graph LR
Regional@{shape: document} --- Controller@{shape: document}
Controller --- Intersection@{shape: document}
```
## Regional
Configurations which are typically the same for all controllers in a region, often due to natioanl/regional regulation.

- default red-yellow time

## Controller
Configuration of a particular controller.
A controller manages one or more physical/logical intersections.

 - Controller id, name, description
 - List of intersections

## Intersection
Configurations for a particular physical/logical intersection.

- Configuration of of signal groups, including things like red-yellow and fixed yellow times.
- Conflict matrix


