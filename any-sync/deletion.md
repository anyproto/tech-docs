## Deletion Process Documentation

The following documentation outlines the process of initiating and handling the deletion of space within the network. This process involves various steps, messages, and actions taken by different nodes within the network.
![Overview](../assets/deletion.png)

- **Deletion Command Initiation:**
    - The deletion command is initiated on the client's side.
    - The command is then sent to the coordinator node for further processing.

- **Space Removal Duration:**
    - Upon receiving the deletion command, the space is scheduled for removal from the network.
    - The removal process is set to occur over a period of 30 days.

- **Deletion Log and Responsible Nodes:**
    - The responsible nodes for the space receive messages from the deletion log.
    - These nodes listen to the deletion log using a poll mechanism.

- **Deletion log: initial "RemovePrepare" Message:**
    - When a responsible node receives the "RemovePrepare" message:
        - The node takes action to prevent the space from being accessed or synced.
        - This marks the beginning of the space removal process.

- **Deletion log: user Revocation Within 30 Days:**
    - During the 30-day removal period, the user has the option to revoke the deletion command.
    - If the user chooses to revoke, the space will continue syncing as before.

- **Deletion log: completion of 30-Day Period:**
    - After the 30-day removal period expires, responsible nodes receive another "RemovePrepare" message in the deletion log:
        - Storage for the space is cleared, preventing further access.
        - The deletion log is removed from the consensus nodes.

- **Notification of Missing Space:**
    - Following the completion of the deletion process, when we try to get the space we will get "space is missing" error.

- **Attempts to Re-add the Space:**
    - If an attempt is made to push the same space to the coordinator:
        - An error message is generated, indicating that the space was previously deleted.

- **Future Considerations:**
    - In the future, a force re-add mechanism can be implemented to reintroduce a previously deleted space.
