# RoboCup@Home 2026 OPL - Rule Book & FAMBOT Implementation

Welcome to the Official Rule Book and Implementation Repository for **FAMBOT** competing in the RoboCup@Home 2026 Open Platform League (OPL).

## Competition Overview
- **Competition**: RoboCup 2026
- **League**: RoboCup@Home – Open Platform League (OPL)
- **Rulebook Version**: 2026 Rev-1 (Final version for RoboCup 2026)
- **Last Revision Date**: May 11, 2026

## About This Repository
This repository contains the competition guidelines, technical specifications, and the official state-machine implementation for FAMBOT's challenge routines.

## Table of Contents
- [General Rules](#general-rules)
- [Technical Specifications](#technical-specifications)
- [Active Challenge: Section 5.1 Receptionist](#active-challenge-section-5-1-receptionist)
- [Safety Guidelines](#safety-guidelines)
- [Scoring System](#scoring-system)

## General Rules
- All teams must comply with the official RoboCup@Home regulations.
- Robots must operate autonomously unless manual interaction is explicitly requested by a task's procedure.
- Robots must wait for a clear start signal (e.g., a start button or voice command) before executing tasks.

## Technical Specifications
- **Robot Platform**: Open Platform League (OPL) compliant.
- **Software Architecture**: Powered by ROS (Robot Operating System).
- **Vision Core**: YOLOv8-pose for human interaction and specialized deep learning vision models for asset/seating classification.
- **Cognition & Audio**: Integrated state-of-the-art Large Language Models (LLMs) via API to handle robust natural language understanding without confirmation questions.

---

## Active Challenge: Section 5.1 Receptionist

This repository primarily houses the module code matching the **Human-Robot Interaction Challenge (Receptionist)** parameters set by the 2026 Technical Committee.

### 1. Task Objective
The robot acts as a party host's receptionist. It must independently:
1. Detect arriving guests at the door via doorbell acoustic cues or manual door opening.
2. Welcome two distinct newcomers, safely extracting their names and favorite drink preferences without requesting manual confirmation or error corrections.
3. Escort guests directly into the living room environment and guide them to valid, unoccupied seating.
4. Introduce the newly arrived guests to each other while maintaining natural human-robot interaction gaze vectors.
5. Accept and transport a luggage piece (bag) handed over by the second guest to an arbitrary designated room location.

### 2. Implementation Execution Flow
The task logic is controlled via a sequential multi-state controller (`mission12026fablablastversion7.py`):
- **States 0–4**: Entrance monitoring, pose identification of Guest 1, and LLM-driven speech extraction.
- **States 5–10**: Navigation through structural waypoints to the living room, point-cloud empty seat detection, and return routing.
- **States 11–15**: Guest 2 profile matching, bag-handover posture registration, and primary voice interaction.
- **States 16–21**: Core spatial re-scanning, dynamic seat allocation for Guest 2, and transition into a high-frequency human tracking follow-mode.

---

## Safety Guidelines
- **Mandatory E-Stop**: The robot must feature an easily accessible, highly visible physical emergency stop button.
- **Collision Management**: Avoidance of major collisions is strictly monitored. Any contact that shifts heavy furniture or risks human safety leads to immediate disqualification from the attempt.

## Scoring System
Points are strictly distributed based on official performance metrics:
- **Main Goals**: Successfully welcoming both guests, offering seats, maintaining gaze interaction, and executing bag handling.
- **Bonus/Additional Goals**: Automatic doorbell sound detection, manual/automated door manipulation, detailed physical visual descriptions of guests, and direct natural language intake without correction prompts.

## Official Links
- **Official RoboCup Website**: [https://www.robocup.org/](https://www.robocup.org/)
- **RoboCup@Home Official Site**: [https://athome.robocup.org/](https://athome.robocup.org/)
- **Official Rulebook Citation**: Hart, Justin et al. *"RoboCup@Home 2026: Rules and regulations"*

## Document Information
- **Maintained By**: FAMBOT Team
- **Version**: 1.0 (Aligned with Official 2026 Rev-1 Rulebook)
- **Last Updated**: 2026-05-28
