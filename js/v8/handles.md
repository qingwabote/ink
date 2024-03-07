[Globals are moveable and not copyable, similar to a unique_ptr.](https://groups.google.com/g/v8-users/c/uSx4F8Uvwis/m/Dvlz55KuAAAJ)
```cpp
// UniquePersistent is an alias for Global for historical reason.
template <class T>
using UniquePersistent = Global<T>;
```