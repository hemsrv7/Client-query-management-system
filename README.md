# Client-query-management-system
The Client Query Management System follows a three-tier architecture.
The presentation layer is built using Streamlit, which provides the user interface for registration, login, query submission, and admin dashboard.
The application layer is implemented in Python and handles business logic such as authentication, validation, role-based access, and query processing.
The data layer uses a MySQL database to store user details and client queries securely.

This architecture ensures proper separation of concerns, easy maintenance, and scalability.

Wireframe / Screen Flow Description
	1.	Home Page
Displays the application title with options to Register or Login.
	2.	Registration Page
New users enter username, password, and role (Client/Support). Data is stored in the users table.
	3.	Login Page
Registered users authenticate using username and password. Session state is created after successful login.
	4.	Client Query Page
Clients submit queries by entering email, mobile number, query heading, and description. Query details are saved in the database with status and timestamps.
	5.	Admin Dashboard
Support users can view all client queries, update query status, and close queries.

Data Flow Description
User input → Streamlit UI → Python backend logic → MySQL database → Response displayed on UI.

User input → Streamlit UI → Python backend logic → MySQL database → Response displayed on UI.
