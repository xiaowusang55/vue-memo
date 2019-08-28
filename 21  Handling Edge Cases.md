# Handling Edge Cases

This page assumes you’ve already read the Components Basics. Read that first ifyou are new to components.

All the features on this page document the handling of edge cases, meaning unusual situations that sometimes require bending Vue’s rules a little. Note however, that they all have disadvantages or situations where they could be dangerous. These are noted in each case, so keep them in mind when deciding to use each feature.

## Element & Component Access

In most cases, it’s best to avoid reaching into other component instances or manually manipulating DOM elements. There are cases, however, when it can be appropriate.

### Accessing the Root Instance

