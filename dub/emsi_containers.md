# EMSI Containers

[emsi_containers](https://github.com/dlang-community/containers) を試す。

## {SourceCode}

```d
/+dub.sdl:
dependency "emsi_containers" version="~>0.7"
+/
import std.stdio;
void main(string[] args)
{
    import containers;
    DynamicArray!int arr;
    arr ~= 1;
    foreach (e; arr)
        e.writeln;
}
```
