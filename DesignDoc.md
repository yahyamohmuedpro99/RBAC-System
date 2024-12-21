# Design Document: Workspace-Based Permission System for TeamApp

## Introduction

This document outlines the enhanced permission system for TeamApp, incorporating workspaces as a core organizational unit. The design builds upon the Hybrid RBAC + Resource Ownership model by adding workspace-level context and permissions.

---

## Understanding Workspaces

A **workspace** acts as a logical grouping for:
1. **Projects**: Projects belong to a specific workspace
2. **Users**: Users are members of a workspace and inherit permissions scoped to that workspace
3. **Permissions**: Permissions can be managed at the workspace level, impacting all resources (projects, tasks) within the workspace

---

## Permission Model with Workspaces

### 1. Roles and Permissions at the Workspace Level
- Roles (`Admin`, `Moderator`, `User`) are assigned within the context of a workspace
- Global permissions are still tied to roles but scoped to the workspace they belong to

### 2. Resource Ownership
- Users are linked to resources (projects, tasks) within a workspace
- Access is determined by their role in the workspace and their ownership of specific resources

---

## Database Design

### 1. Workspaces Collection
```json
[
  {
    "_id": "workspace_123",
    "name": "Engineering Workspace",
    "members": [
      {"userId": "user_1", "role": "admin"},
      {"userId": "user_2", "role": "moderator"},
      {"userId": "user_3", "role": "user"}
    ]
  }
]
```

### 2. Projects Collection
```json
[
  {
    "_id": "project_456",
    "name": "Project Alpha",
    "workspaceId": "workspace_123",
    "team": ["user_2", "user_3"]
  }
]
```

### 3. Tasks Collection
```json
[
  {
    "_id": "task_123",
    "name": "Fix Bug",
    "projectId": "project_456",
    "workspaceId": "workspace_123",
    "assignedTo": "user_3"
  }
]
```

### 4. Roles Collection
```json
[
  {
    "_id": "role_admin",
    "name": "admin",
    "permissions": [
      {"action": "crud", "scope": "project"},
      {"action": "crud", "scope": "task"},
      {"action": "manage", "scope": "workspace"}
    ]
  },
  {
    "_id": "role_moderator",
    "name": "moderator",
    "permissions": [
      {"action": "read", "scope": "project"},
      {"action": "crud", "scope": "task"}
    ]
  },
  {
    "_id": "role_user",
    "name": "user",
    "permissions": [
      {"action": "read", "scope": "project"},
      {"action": "read", "scope": "task"}
    ]
  }
]
```

---

## Permission Enforcement with Workspaces

### How It Works
1. **Role-Based Global Permissions within a Workspace**:
   - A user's role in the workspace determines their global permissions

2. **Scoped Resource Ownership**:
   - Membership in a project (team) or assignment to a task provides finer-grained access control

### Middleware Logic
When a user performs an action, the system checks:
1. **Workspace Membership**:
   - Is the user a member of the workspace?
   - What is their role in the workspace?

2. **Role-Based Permissions**:
   - Does the user's role allow the action (e.g., `task_crud`)?

3. **Resource Ownership**:
   - Is the user part of the resource (e.g., project team or task assignee)?

4. **Default Deny**:
   - If none of the above conditions are met, deny access

---

## Benefits of This Model

1. **Supports Workspace Context**:
   - Permissions are scoped to a workspace, ensuring users only access resources they are part of

2. **Maintains Granularity**:
   - Resource ownership (e.g., project team or task assignment) still controls fine-grained access

3. **Simplifies Management**:
   - Assigning roles at the workspace level avoids excessive scoped permissions for each resource

4. **Scalable**:
   - Workspaces create a natural hierarchy, making it easy to add new users, projects, or tasks without complicating access control

---

## Concerns Addressed

1. **Complexity**:
   - Adding workspaces may seem complex, but they provide a natural grouping for resources and users, simplifying overall permission management

2. **Scalability**:
   - Workspace-based roles prevent permission explosion while still supporting dynamic access control

3. **Performance**:
   - Workspace membership and resource ownership checks are efficient and leverage indexed queries

---
