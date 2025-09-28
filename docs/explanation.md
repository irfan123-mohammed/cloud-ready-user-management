# Detailed Explanation of `user_management.sh`

This script is a **menu-driven Bash program** that helps Linux administrators (or Cloud Support Associates) manage users efficiently.  
It covers the following operations:
- Creating users
- Deleting users
- Listing users
- Locking/unlocking users

Let’s break it down step by step.

---

## 1. Root Privilege Check
```bash
check_root() {
    if [[ "$EUID" -ne 0 ]]; then
        echo "Error: This script must be run as root." >&2
        exit 1
    fi
}

*$EUID holds the effective user ID of the person running the script.

*Root’s ID is always 0.

*If the script is not run as root → it prints an error and exits.

*This prevents unauthorized changes to system users.

2. Create User Function

create_user() {
    read -p "Enter username to create: " username
    if id "$username" &>/dev/null; then
        echo "User '$username' already exists."
        return
    fi

*Prompts for a username.

*id username checks if the user already exists.

*If yes, it stops execution.

    read -s -p "Enter password for $username: " password
    echo
    useradd -m -s /bin/bash "$username"
    echo "$username:$password" | chpasswd
    echo "User '$username' created successfully."

*Prompts for a password (hidden input, -s).

*useradd -m -s /bin/bash creates the user with a home directory and Bash shell.

*chpasswd sets the password.

*Prints success message.

    read -p "Add user to a group? (y/n): " add_group
    if [[ "$add_group" == "y" ]]; then
        read -p "Enter group name: " groupname
        if grep -q "^$groupname:" /etc/group; then
            usermod -aG "$groupname" "$username"
            echo "User '$username' added to group '$groupname'."
        else
            echo "Group '$groupname' does not exist."
        fi
    fi
}


*Asks whether to add the user to an existing group.

*grep "^groupname:" /etc/group checks if the group exists.

*If yes → usermod -aG adds the user to that group.

3. Delete User Function

delete_user() {
    read -p "Enter username to delete: " username
    if ! id "$username" &>/dev/null; then
        echo "User '$username' does not exist."
        return
    fi

* Checks if the user exists before deleting

    read -p "Are you sure want to delete user '$username'? (y/n): " confirm
    if [[ "$confirm" == "y" ]]; then
        userdel -r "$username"
        echo "User '$username' deleted successfully."
    else
        echo "User deletion aborted."
    fi
}

*userdel -r deletes both the user account and their home directory.

*Asks for confirmation to avoid accidental deletion.

4. List Users Function

list_users() {
    echo "Listing all system users:"
    awk -F':' '{ print $1 }' /etc/passwd
}

*/etc/passwd contains all user accounts.

*awk -F':' '{ print $1 }' prints only the username field.

5. Lock/Unlock User Functions

lock_user() {
    read -p "Enter username to lock: " username
    if id "$username" &>/dev/null; then
        passwd -l "$username"
        echo "User '$username' has been locked."
    else
        echo "User '$username' does not exist."
    fi
}


*Locks the account (disables login).

*Achieved by putting a ! before the password in /etc/shadow.

unlock_user() {
    read -p "Enter username to unlock: " username
    if id "$username" &>/dev/null; then
        passwd -u "$username"
        echo "User '$username' has been unlocked."
    else
        echo "User '$username' does not exist."
    fi
}

*Unlocks the user account.


6. Menu Function

show_menu() {
    echo "..........................."
    echo "Ubuntu User Management Script "
    echo "..........................."
    echo "1) Create a new user"
    echo "2) Delete a user"
    echo "3) List all users"
    echo "4) Lock a user"
    echo "5) Unlock a user"
    echo "6) Exit"
    echo "............................"
}

*Displays a simple menu to guide the user.

7. Main Program

check_root

while true; do
    show_menu
    read -p "Choose an option: " choice

    case $choice in
        1) create_user ;;
        2) delete_user ;;
        3) list_users ;;
        4) lock_user ;;
        5) unlock_user ;;
        6) echo "Exiting..."; exit 0 ;;
        *) echo "Invalid option. Please select a valid choice." ;;
    esac
done


*Calls check_root first.

*Starts an infinite loop.

*Reads user input and calls the correct function using case.

*Exits only if option 6 is chosen.

