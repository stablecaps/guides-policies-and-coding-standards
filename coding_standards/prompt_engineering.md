# Prompt Engineering

## Overview

prompts for Github copilot

## Docstring
Generate google-style docstrings for highlighted code. Also include examples

## Tests
@workspace /tests generate tests for selected function. use /tmp directory to create files and directories if required. Clean up any test artifacts created. if the test fails, the failure message should cearly indicate why the test failed. Ensure that function being tested is called with all arguments even if it has defaults. these are test cases: ...


When multiple tests are present fior a function use a pytest class with each test seperated into a different method.
Ensure that assertions have a descriptive error message.
Ensure code is tested for the correct input and output.
Ensure code is tested for the incorrect input and output.
Ensure code is tested for the edge cases.
