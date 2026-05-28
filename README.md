## Section 5.1: Human Robot Interaction Challenge (Receptionist)

### Description
The robot works as a receptionist at a party. Its task is to welcome two new guests, escort them to the living room, offer them a place to sit, and introduce them to each other. The second guest also brings a bag for the robot to carry.

### Goals
* **Main Goal:** Welcome and assist two newcomers at a party and offer them a seat while maintaining an appropriate gaze direction during conversation.
* **Bag Handling:** Handle a bag brought by the second guest to be placed somewhere in the house.

### Additional Goals
* Detect a doorbell sound as a signal that a guest has arrived.
* Open the entrance door for each arriving guest.
* Visually describe the first guest to the second guest.
* Understand each guest's name and favorite drink without asking any confirmation or correction questions.

### Technical Focus
* System Integration
* Human-Robot Interaction (HRI)
* Person Detection
* Person Recognition

### Task Procedure
1. **Doorbell Detection:** The robot waits at the starting position and detects the doorbell sound when a guest arrives.
2. **Greeting & Intake:** The robot either opens the entrance door or meets the guest there, greets them, and asks for their name and favorite drink.
3. **Escorting & Seating:** The robot escorts the guest to the living room and offers them a free seat.
4. **Guest Introduction:** Once both guests are seated, the robot introduces them by looking at one guest and stating the other's name and favorite drink.
5. **Bag Delivery:** The robot requests a bag handover from the second guest, follows the host to a random location in the house, and drops the bag where instructed.
