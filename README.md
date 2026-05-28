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

# Section 5.1: Receptionist Challenge Implementation

This repository contains the implementation of the **RoboCup@Home 2026 Receptionist Challenge (Task 5.1)**. The robot acts as a party receptionist: welcoming two arriving guests, identifying their names and favorite drinks, escorting them to the living room, identifying available seating using computer vision, and finally following the host.

## Core Module Structure & Key Functions

The implementation in `mission12026fablablastversion7.py` is divided into several specialized functional components:

### 1. Motion and Navigation Controls
* `move(dis, turn)`: Publishes linear and angular velocity commands (`geometry_msgs/Twist`) to the chassis driver topic `/cmd_vel`.
* `turn(angle)` / `turn_to(angle, speed)`: Controls precise rotational movements utilizing odometry feedback.
* `turn_back(speed)`: Executes a 180-degree rotation relative to the current heading.

### 2. Vision and Human Perception
* `find_user(img, depth)`: Deploys `yolov8n-pose.pt` to detect human keypoints and calculate their proximity using a depth map.
* `find_user_seated(image, depth)`: A specialized algorithm optimized to locate and track seated individuals by evaluation of bounding boxes, spatial coordinates, and depth variance.
* `get_person_feature_seated(image, user)` / `get_person_feature(...)`: Sends cropped imagery of human targets to the OpenRouter Vision API (`qwen/qwen3.5-flash-02-23`) to analyze specific physical descriptors (e.g., clothing colors, glasses, hats).
* `classify_person(...)` / `feature_score(...)`: Compares current visual feature profiles against pre-recorded benchmarks to differentiate between **Guest 1** and the **Host**.

### 3. Audio & Natural Language Processing
* `wait_for_new_audio(timeout)`: Listens for incoming speech audio data paths received via ROS.
* `audio_extract_name(...)` / `audio_extract_drink(...)`: Transcribes and processes auditory files using the multi-modal LLM (`google/gemini-2.5-flash-lite`) via OpenRouter, matching results safely to standardized data subsets (`NAME_LIST` and `DRINK_LIST`).

### 4. Baggage Interaction
* `bag_any_xy(image)` / `is_bag_handover_ready(...)`: Utilizes a custom fine-tuned model (`bag_model_2026.pt`) to detect luggage orientation, coordinates, and depth limits for a safe handover procedure.

### 5. Interactive Human-Following
* `follow()`: Initiates a responsive real-time tracker loop. The robot tracks human coordinates (`find_human_xy`) and dynamically adjusts velocities to maintain a set distance of 700mm, stopping abruptly if the operator declares "stop".

---

## State Machine Execution Flow

The main function runs a sequential state machine tracking the completion of sub-goals:

```mermaid
graph TD
    S0[State 0: Ready Initialization] --> S1[State 1: Scan & Find Guest 1]
    S1 --> S2[State 2: Navigate to Guest 1 Entry Point]
    S2 --> S25[State 25: Adjust Orientation]
    S25 --> S3[State 3: Prompt & Audio Extraction for Name]
    S3 --> S4[State 4: Prompt & Audio Extraction for Drink]
    S4 --> S5[State 5: Announce 'Please Follow Me']
    S5 --> S6[State 6: Pivot 180 Degrees]
    S6 --> S7[State 7: Traverse to Waypoint 1]
    S7 --> S8[State 8: Advance to Living Room Main Coordinates]
    S8 --> S9[State 9: Rotational Scan for Open Seats Left/Middle/Right/Far Right]
    S9 --> S10[State 10: Face Front Forward Position]
    S10 --> S105[State 105: Return to Entrance for Second Target]
    S105 --> S11[State 11: Detect & Capture Features of Guest 2]
    S11 --> S12[State 12: Move to Interaction Space]
    S12 --> S125[State 125: Turn to Face Guest 2]
    S125 --> S13[State 13: Extract Guest 2 Name]
    S13 --> S14[State 14: Extract Guest 2 Drink preference]
    S14 --> S155[State 155: Order Bag Handover Alignment]
    S155 --> S15[State 15: Announce Escort Route Start]
    S15 --> S16[State 16: Turn Around]
    S16 --> S17[State 17: Return Through Waypoint 1]
    S17 --> S18[State 18: Re-enter Living Room Hub]
    S18 --> S19[State 19: Full Suite Scan: Identify Remaining Empty Seats and Locate Guest 1]
    S19 --> S20[State 20: Direct Guest 2 to Free Seating Area]
    S20 --> S21[State 21: Align with Host Position & Trigger follow Module]
