# Detailed Implementation Plan for PR Splitter MCP Server

This document outlines the detailed implementation plan for creating an MCP server that leverages Graphite's `gt` CLI to support branching and PR creation functionalities in a PR splitting workflow.

---

## 1. Overview

The goal is to build an MCP server that exposes two separate tools:
- **create_branch**: Creates a new branch using Graphite's `gt create` command with a logical naming scheme.
- **create_pr**: Creates a pull request, or opens the PR page, using Graphite's `gt pr` command.

The branch naming convention will generate names based on the base branch (or PR) identifier with logical sequencing (e.g., `PR1-A`, `PR1-B`, `PR1-B1`, etc.).

---

## 2. Architecture

**Language & Framework:**  
- Node.js with TypeScript.

**Project Structure:**  
- The MCP server project will follow a structure similar to the example weather server:
  - `src/`: Contains server implementation code.
  - `tsconfig.json`: TypeScript configuration.
  - `package.json`: Project configuration.
  
**Key Components:**  
- **Server Initialization:**  
  - Configure and initialize the MCP server instance.
  - Define error handlers and shutdown procedures.

- **Tool Handlers:**  
  - Separate request handlers for each tool (`create_branch` and `create_pr`).

- **Command Execution Module:**  
  - A utility module to execute Graphite `gt` CLI commands using Node's child process functionalities.
  - Capture stdout, stderr, and error codes to relay meaningful error messages.

---

## 3. Tool Definitions

### 3.1. create_branch Tool
- **Purpose:**  
  Create a new branch using the `gt create` command.
  
- **Input Schema:**  
  - `baseName` (string): Base branch or PR identifier (e.g., "PR1").
  - `sequence` (optional string): Logical sequence detail (e.g., "A", "B", etc.); if not provided, the server should compute the next sequence based on previous calls.

- **Operation:**  
  - Compute new branch name based on base name and sequence.
  - Execute `gt create` with the computed branch name.
  - Capture and return command output.

### 3.2. create_pr Tool
- **Purpose:**  
  Create a pull request for a specified branch using the `gt pr` command.
  
- **Input Schema:**  
  - `branch` (string): Target branch for which the PR is to be created.
  - Optional parameters for PR title and description may be added later.

- **Operation:**  
  - Execute `gt pr` for the given branch.
  - Alternatively, use `gt submit` if further automation is desired.
  - Return PR URL or status message from the command output.

---

## 4. Branch Naming Strategy

- **Naming Convention:**  
  - Base name provided (ex: "PR1") concatenated with a sequential letter/number.  
  - Examples:
    - First split: `PR1-A`
    - Second split: `PR1-B`
    - Further splits on branch PR1-B: `PR1-B1`, `PR1-B2`, etc.
  
- **Implementation:**  
  - Develop a simple algorithm to track previous splits per base branch.
  - Consider storing counts temporarily during server runtime, or compute based on existing branch names via a Graphite command if needed.

---

## 5. Command Execution Details

- **Environment:**  
  - Ensure that the `gt` CLI is installed and available on the system where the MCP server runs.
  
- **Execution Strategy:**  
  - Use Node's `child_process.exec` (or similar) to run `gt` commands.
  - Capture both stdout and stderr.
  - Return parsed command output to the MCP tool caller.
  
- **Error Handling:**  
  - Process exit codes should be checked.
  - Any errors should be formatted and sent back with clear error messages.

---

## 6. Development and Testing Process

1. **Scaffold the MCP Server Project:**
   - Use `npx @modelcontextprotocol/create-server` to initialize the project.
   - Set up TypeScript and necessary dependencies.

2. **Implement Command Execution Utilities:**
   - Create helper functions to execute system commands and handle results.

3. **Implement Tool Handlers:**
   - Set up request handlers for `create_branch` and `create_pr` tools.
   - Integrate branch naming strategy and invoke the CLI commands accordingly.

4. **Testing:**
   - Write unit tests and integration tests to simulate tool requests.
   - Test command execution in a controlled development environment.
   - Validate proper error handling and branch naming outputs.

5. **Documentation:**
   - Update internal documentation (e.g., this plan).
   - Ensure clear usage instructions for the LLM on tool functionalities.

---

## 7. Future Enhancements

- **Enhanced Input Schemas:**  
  - Expand PR creation tool with additional parameters (title, description, draft mode, etc.).
  
- **Persistent State Management:**  
  - Implement persistent tracking of branch splits for more robust naming conventions.
  
- **Additional Tools:**  
  - Integrate further Graphite CLI commands (e.g., for branch restacking, merging, etc.) if needed.

---

This implementation plan provides a clear roadmap for developing the PR Splitter MCP server with integrated Graphite CLI functionalities.
