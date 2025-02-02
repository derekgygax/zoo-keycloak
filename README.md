# Keycloak

## Core Concepts

A realm secures and manages security metadata for a set of users, applications, and registered oauth clients. 
Users can be created within a specific realm within the Administration console. 
Roles (permission types) can be defined at the realm level 
and you can also set up user role mappings to assign these permissions to specific users.

- **Realm**: A zoo branch (e.g., "Central Zoo") that manages its own animals, staff, and visitors separately from other branches.
- **Role**: A job title at the zoo (e.g., "Vet", "Cleaner", "Admin") that defines what someone can do.
- **User**: A person at the zoo (e.g., John, the vet) who is assigned a role and given permissions.
- **Client**: The zoo’s website or app (e.g., "zoo-management") that people use to log in. It doesn’t handle authentication itself but relies on Keycloak to verify users and check their roles before granting access.



---

## Download Keycloak  
Visit the official [Keycloak Downloads](https://www.keycloak.org/downloads) page.  
For this setup, download:  
- **Server** → **Keycloak (Quarkus Distribution)** → **ZIP Format**

To understand the contents of your Keycloak installation, see the [directory structure guide](https://www.keycloak.org/server/directory-structure).

## Setup Instructions

### 1. Start Keycloak
```sh
bin/kc.sh --http-port=8100
```
DEV mode
```sh
bin/kc.sh start-dev --http-port=8100
```

After the server boots, open http://localhost:8100 in your web browser. The welcome page will indicate that the server is running.

Docs configuration guide [configuration guides](https://www.keycloak.org/guides#server).

### 2. Create a Temporary Admin  
Go through the setup steps to access the admin panel.

### 3. Create a Permanent Admin  
- Click **Users** on the left.
- Click **Add User** and fill in:
  - **Email Verified**: ON
  - **Username**: admin
  - **First Name**: admin
  - **Last Name**: admin
- Click **Create**.
- Go to the **Credentials** tab:
  - Set a password.
  - Turn **Temporary** OFF.
  - Save the password.
- Go to the **Role Mapping** tab:
  - Click **Assign Role**.
  - In the top-left dropdown, change **Filter by clients** → **Filter by Realm Roles**.
  - Assign:
    - `admin`
    - `create-realm`
- Click **Assign**.
- Log out and sign in as the new admin.
- Delete the temporary admin.

### 4. Create a Realm
- Click the realm dropdown in the top left.
- Click **Create Realm**.
- Set the name to `zoo`.
- Click **Create**.

### 5. Add the Client
- Click **Clients**.
- Click **Create Client**.
  - **Client Type**: OpenID Connect
  - **Client ID**: `zoo-client`
  - **Name**: `Zoo Client`
- Click **Next**.
  - **Client Authentication**: ON
  - **Authorization**: OFF
  - **Standard Flow**: CHECK
  - **Direct Access**: CHECK
  - All others: UNCHECK
- Click **Next**.
  - **Root URL**: `http://localhost:3000`
  - **Home URL**: `http://localhost:3000`
  - **Redirect URIs**:  
    - `http://localhost:3000/api/auth/callback`
  - **Logout Redirect URIs**:
    - `http://localhost:3000/`
  - **Web Origins**:
    - `http://localhost:3000`
- Click **Save**.

### 6. Add Roles
- Click **Realm Roles**.
- Click **Create Role** and add:
  - `GUEST`
  - `ADMIN`
  - `STAFF`
  - `VET`
  - `VENDOR`

### 7. Configure Google Login
- Click **Identity Provider** > **Google**.
- In **Google Cloud Console**:
  - Go to **APIs & Services** > **Credentials**.
  - Select **Zoo Management**.
  - Add **Authorized JavaScript Origins**:
    - `http://localhost:8100`
  - Add **Authorized Redirect URIs**:
    - `http://localhost:8100/realms/zoo/broker/google/endpoint`
  - Save changes.
- In **Keycloak**:
  - **Alias**: `google`
  - **Client ID**: from Google (`.env`)
  - **Client Secret**: from Google (`.env`)
- Click **Add**.

### 8. Set `GUEST` as Default Role
- Click **Identity Provider** > **Google**.
- Go to the **Mapper** tab.
- Click **Add Mapper**.
  - **Name**: `Default Role GUEST`
  - **Sync Mode Override**: Import
  - **Mapper Type**: Hardcoded Role
  - **Select Role**:
    - Change top-left dropdown **Filter by Realm Role**.
    - Select **GUEST**.
    - Click **Assign**.
- Click **Save**.

### 9. Enable Auto User Creation for Google Login
- Click **Identity Provider** > **Google**.
- Scroll to the bottom:
  - **First Login Flow Override**: `first broker login`
  - **Trust Email**: ON
- Click **Save**.
- Go to **Realm Settings** > **Login**.
  - **User Registration**: ON.
  - **Login with Email**: (default ON, adjust if needed).

### 10. Get the Client Secret
- Click **Clients** > **zoo-client**.
- Scroll down:
  - **Client Authentication**: ON
- Click **Save**.
- Click **Credentials** tab.
- Copy the **Client Secret**.
- Store in `.env` in next.js zoo as:
  ```env
  KEYCLOAK_CLIENT_SECRET=your_client_secret_here
  ```

### 11. Assign Roles to Users
- Click **Users**.
- Select a user.
- Click **Assign Role**.
- In the pop-up, change the top-left dropdown to **Filter by Realm Roles**.
- Select the roles.
- Click **Assign**.