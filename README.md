# RoboCup@Home 2026 OPL - Rule Book

Welcome to the Official Rule Book Repository for RoboCup@Home 2026 Open Platform League (OPL) and unified leagues.

## Competition Overview
- **Competition**: RoboCup 2026
- **League**: @Home Open Platform League (OPL)
- **Location**: Incheon, Republic of Korea
- **Date**: June 30 - July 6, 2026

## About This Repository
This repository contains the official rule book, competition guidelines, technical specifications, and regulations for the RoboCup@Home 2026 OPL competition maintained by the Technical Committee.

## Table of Contents
- [General Rules](#general-rules)
- [Technical Specifications](#technical-specifications)
- [Test Procedures](#test-procedures)
- [Safety Guidelines](#safety-guidelines)
- [Team Requirements](#team-requirements)
- [Scoring System](#scoring-system)
- [Document Structure](#document-structure)
- [Important Links](#important-links)
- [Version Information](#version-information)
- [Questions & Support](#questions--support)

## General Rules
- All teams must comply strictly with the official RoboCup@Home regulations.
- Robots must operate completely autonomously unless manual human interaction or coaching is explicitly authorized in a specific task description.
- Robots must wait at their designated starting locations for a clear, official start signal (e.g., physical start button, verbal greeting, or external sensor trigger).
- Fair play, mutual assistance, and sportsmanship are mandatory across all teams and arena testing schedules.

## Technical Specifications
- **Robot Platform**: Open Platform League compliant (teams design or select any custom or commercial hardware architecture).
- **Communication & Protocols**: Standard middleware setups (such as ROS or ROS2), utilizing localized processing nodes. External internet/network API dependencies must comply with the committee's real-time wireless connectivity parameters.
- **Sensors**: Any combination of localized RGB-D cameras, LiDAR modules, IMUs, microphones, and touch arrays within functional safety bounds.
- **Power & Actuation**: Independent on-board battery systems. Manipulator configurations and chassis actuators must execute safely near human operators.

## Test Procedures
Refer to the individual test handbooks for exact procedural parameters:
- **Task Specifications**: Step-by-step state benchmarks ranging from Section 4 Domestic/Manipulation skills to Section 5 Social Interaction Challenges.
- **Evaluation Criteria**: Objective validation of sub-tasks, such as recognizing a dynamic human posture, understanding text without prompt corrections, or locating objects.
- **Scoring Methodology**: Concrete task-milestone score splits, detailing point weight distributions for essential actions versus optional elements.
- **Time Limits**: Fixed execution slots (typically 5 to 10 minutes max per attempt) including setup and arena exit parameters.

## Safety Guidelines
- **Mandatory Safety Inspection**: All robots must successfully pass the official technical safety inspection during Setup Days before entering the arena.
- **Emergency Stop (E-Stop)**: A physical, hardwired, highly visible, and easily reachable emergency stop mechanism is mandatory on every robot platform to instantly cut actuator power.
- **Collision Regulations**: Avoiding major collisions is critical. Hitting arena boundaries or dragging heavy furniture results in immediate trial termination and a score of zero.
- **Liability**: Teams are entirely responsible and liable for any structural or physical damage caused by their robots during the competition.

## Team Requirements
- **Preregistration**: Initial registration and documentation profiles must be submitted via the designated channels within scheduled deadlines.
- **Qualification Video**: Teams must provide a valid qualification video demonstrating core navigation, manipulation, or human-robot communication capabilities.
- **Academic Publications**: Submission of a detailed Team Description Paper (TDP) outlining the technical innovations, hardware setups, and software frameworks is required.
- **On-Site Representation**: Registered members must actively attend technical meetings, support infrastructure setup, and sign the official code of conduct.

## Scoring System
- **Main Goal Points**: Points are awarded sequentially based on successful milestone completion (e.g., meeting a guest, correctly picking up a target, navigating to a waypoint).
- **Additional/Bonus Points**: Additional score weights are granted for extraordinary technical accomplishments (e.g., navigating without asking correction prompts or handling un-mapped dynamic obstacles).
- **Penalties & Deductions**: Score penalties or total trial invalidation are given for manual human interventions, safety line boundary violations, or obstructing opponent setups.
- **Open Challenges & Jury Scoring**: Special phases (e.g., Open Challenge or Poster Presentations) are evaluated by a panel of league experts using standardized qualitative scoring guidelines.

## Document Structure
- `general-rules/` — Governance framework, competition foundations, and team codes of conduct.
- `technical/` — Arena environmental layouts, object definitions, and hardware/network rules.
- `tests/` — Procedural handbooks detailing Section 4 (Manipulation) and Section 5 (Human-Robot Interaction) tests.
- `scoring/` — Point sheets, qualification matrix guidelines, and penalty scoring sheets.
- `safety/` — Safety inspection check-lists, physical E-stop requirements, and validation logs.

## Important Links
- **Official RoboCup Website**: [https://www.robocup.org/](https://www.robocup.org/)
- **RoboCup@Home Official Site**: [https://athome.robocup.org/](https://athome.robocup.org/)
- **Rulebook Repository**: [https://github.com/RoboCupAtHome/RuleBook](https://github.com/RoboCupAtHome/RuleBook)
- **RoboCup@Home Mailing List**: [https://lists.robocup.org/](https://lists.robocup.org/)

## Version Information
- **Version**: 2026 Rev-1
- **Last Updated**: 2026-05-11
- **Status**: Official Release / Final Version for RoboCup 2026

## Questions & Support
For rule inquiries, text clarifications, or general technical questions:
1. Check the official repository Wiki or open GitHub Issues inside this project.
2. Submit a formal query to the official RoboCup@Home Technical Committee mailing list.
3. Participate in the designated technical discussion panels held during on-site Setup Days.

---
*Copyright © 2026 RoboCup@Home Technical Committee. All rights reserved.*
*This site is used by FAMBOT*
