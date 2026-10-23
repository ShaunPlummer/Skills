---
name: review-kotlin-coroutines
description: >-
  Reviews documentation no classes, function and  properties to identify improvements.
disable-model-invocation: true
---

# Process
1. Identify the scope of the review process. If the user has not specified, limit the scope of the documentation review to only uncommited changes in the working directory.
2. Follow the 'Documentation Guidance' to update the project documentation for classes, properties and funtions.

# Documentation Guidance

- Use plain language that’s easy to understand.
- Document only necessary information.


Begin the description of entry points with a clear, high-level description of what the function does. Avoid simply restating the signature in natural language.

For example, don't say “takes a String and returns a Connection” but instead perhaps “Attempts to connect to the database specified by the input string, returning a Connection if successful, and throwing a ConnectionTimeoutException otherwise”.

Specify each input's expected values and behavior with different inputs. Explain the range of valid values and what happens when invalid values are provided. For example, if a string input is supposed to be a URL, describe what happens when the string is empty, invalid, uses an unsupported protocol, or refers to a location that does not exist.

Document every exception an API entry point may throw. Discuss failure conditions in the general description and reserve the exception section for detailed information. This enhances readability and helps the reader focus. Instead, include this information organically into the general description. Providing usage examples whenever possible also helps users understand how to use the API correctly.

Document interfaces as contracts, describing the behaviour, inputs, outputs, and guarantees that every implementation must satisfy. Keep this documentation independent of implementation choices, placing details such as storage technologies, algorithms, and internal mechanisms on the concrete implementations. Implementation-specific documentation should clarify how an implementation fulfils the shared contract without contradicting its guarantees.

## Formatting Rules
For longer documentation comments, place the opening /** on a separate line and begin each subsequent line with an asterisk:

/**
 * This is a documentation comment
 * on multiple lines.
 */
Short comments can be placed on a single line:

/** This is a short documentation comment. */
Generally, avoid using @param and @return tags. Instead, incorporate the description of parameters and return values directly into the documentation comment, and add links to parameters wherever they are mentioned. Use @param and @return only when a lengthy description is required which doesn't fit into the flow of the main text.

// Avoid doing this:

/**
 * Returns the absolute value of the given number.
 * @param number The number to return the absolute value for.
 * @return The absolute value.
 */
fun abs(number: Int): Int { /*...*/ }

// Do this instead:

/**
 * Returns the absolute value of the given [number].
 */
fun abs(number: Int): Int { /*...*/ }