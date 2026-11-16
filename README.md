# rules_ros2_plugins

Bazel rules for ROS 2 pluginlib plugins: the `ros2_plugin` macro, its `ros2_plugin_rule`, the
`RosPluginInfo` / `RosPluginCollectorInfo` providers and the `ros2_plugin_collector_aspect`
that launchers use to build the ament index for the plugins a binary needs.

This code used to live in `com_github_mvukov_rules_ros2` (`ros2/plugin.bzl` and the plugin part
of `ros2/plugin_aspects.bzl`). It was moved into its own module so that packages that only
build plugins (urdf, nav2, diagnostics, image_common, rosbag2, ...) no longer depend on
`rules_ros2`, and through it on `ros2cli`, launch and the rest of the CLI stack. It depends only
on `rules_cc`.

## What changed

`ros2_plugin` now builds the plugin implementation with a plain `rules_cc` `cc_library` instead of
`rules_ros2`'s `ros2_cpp_library`. That macro only added `-std=c++17 -fPIC` and a
`ROS_PACKAGE_NAME` local define, so set the C++ standard through your toolchain or `copts` if
you relied on it. Everything else is unchanged.

## What moved, and compatibility

| Was in `com_github_mvukov_rules_ros2` | Now | Old label still works? |
|---|---|---|
| `//ros2:plugin.bzl` (`ros2_plugin`, `ros2_plugin_rule`) | `@rules_ros2_plugins//ros2:plugin.bzl` | Yes: the old file re-exports them |
| `RosPluginInfo`, `RosPluginCollectorInfo`, `ros2_plugin_collector_aspect`, `create_dynamic_library` in `//ros2:plugin_aspects.bzl` | `@rules_ros2_plugins//ros2:plugin_aspects.bzl` | Yes: re-exported |

Providers and the aspect are re-exported, not copied, so plugins built through either label are
found by `rules_ros2`'s launchers (`ros2_launch`, `set_up_ament`, tests). The interface aspects
(`ros2_interface_collector_aspect`, `ros2_idl_plugin_aspect`) stay in `rules_ros2`.

## Usage

```starlark
bazel_dep(name = "rules_ros2_plugins", version = "0.1.0")
```

```starlark
load("@rules_ros2_plugins//ros2:plugin.bzl", "ros2_plugin")

ros2_plugin(
    name = "my_plugins",
    srcs = ["src/my_plugin.cpp"],
    plugin_specs = [
        {
            "class_type": "my_pkg::MyPlugin",
            "base_class_type": "base_pkg::Base",
        },
    ],
    deps = ["@ros2_pluginlib//:pluginlib"],
)
```

Put the plugin target in the `data` of the binary that loads it. The ament index that pluginlib
reads at runtime comes from `rules_ros2`'s launch wrappers, which use the collector aspect.

## License

Apache 2.0, same as rules_ros2, which this code comes from. See [LICENSE](LICENSE).
