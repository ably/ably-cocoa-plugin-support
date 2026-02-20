# Ably Cocoa SDK Plugin Support Library

This is an Ably-internal library that defines the private APIs that the [Ably Pub/Sub Cocoa SDK](https://github.com/ably/ably-cocoa) uses to communicate with its plugins.

> [!note]
> This library is an implementation detail of the Ably SDK and should not be used directly by end users of the SDK.

Further documentation will be added in the future.

## Conventions

- Methods whose names begin with `nosync_` must always be called on the client's internal queue.

## Dependency setup

The intention is that:

- ably-cocoa's `Package.swift` declares a dependency on ably-cocoa-plugin-support
- each plugin's `Package.swift` declares a dependency on ably-cocoa-plugin-support
- each plugin's `Package.swift` also declares a direct dependency on ably-cocoa

This means that:

- a given version of the plugin is able to specify a _minimum_ version of ably-cocoa
- a given version of ably-cocoa is expected to work with all plugin versions whose minimum it satisfies — i.e. ably-cocoa is _not_ able to specify a minimum version of a plugin

The only way that ably-cocoa can exclude certain plugin versions is by creating a clash in the major version of the ably-cocoa-plugin-support dependency; we avoid using this mechanism where possible as it can cause confusing dependency resolution error messages for users and complicates the release process.

## Playbook for making changes

This section describes how to make some common changes that require changes to all three repositories (ably-cocoa, the plugin, and this repo).

### Adding new capabilities

In this section we describe how to make _non-breaking plugin-support changes_ (i.e. a minor plugin-support version bump) that either introduce functionality only available in a new version of ably-cocoa, or introduce functionality only available in a new version of the plugin.

The key point is that both of the following are breaking plugin-support changes and thus must be avoided:

- adding a new required method to an Objective-C protocol (breaks implementers of that protocol)
- removing a required method from an Objective-C protocol (breaks consumers of that protocol)

#### Introducing new functionality only available in a new version of ably-cocoa

1. Add a new `@optional` method or property.
2. Increase the plugin's minimum ably-cocoa version so that this method is guaranteed to be implemented, and perform a runtime precondition failure if it is not.

> [!NOTE]
> Avoid introducing new `@optional` properties with a nullable type, since in this case Swift does not provide a mechanism for differentiating between "property not implemented" and "property implemented and its value is `nil`". Use a getter method in this situation instead.

#### Introducing new functionality only available in a new version of the plugin

1. Add a new `@optional` method or property.
2. Perform a `-respondsToSelector:` check in ably-cocoa to decide whether this new API can be called. ably-cocoa **must** still behave correctly in the case where it cannot (i.e. where an old version of the plugin is being used).

### Removing capabilities

In this section we describe how to make _non-breaking plugin-support changes_ (i.e. a minor plugin-support version bump) that either remove functionality from ably-cocoa, or remove functionality from the plugin.

This also covers the case where, in combination with [adding a new capability](#adding-new-capabilities), we are replacing a method with a new variant.

The key point is that it is a breaking plugin-support change to remove a method from an Objective-C protocol (breaks consumers of that protocol) and this must thus be avoided.

#### Removing functionality from the plugin

Since the plugin can exclude older versions of ably-cocoa, the plugin is able to choose to throw a runtime exception in methods that it no longer wishes to implement. This should be combined with increasing the plugin's minimum ably-cocoa version to one that no longer calls the method, so that the exception is never triggered in practice.

#### Removing functionality from ably-cocoa

Since new versions of ably-cocoa can be used with old versions of the plugin, ably-cocoa must continue to provide functioning implementations of all methods.
