---

# English-Learning Robot Using ROS2, LLM, and TTS

## Overview

This repository provides the setup and implementation steps for creating an interactive English learning 
robot using **ROS2**, **Language Models (LLM)**, and **Text-to-Speech (TTS)** technologies. 
The robot processes user input, generates meaningful responses using an LLM, 
and converts those responses into speech using TTS.

## Table of Contents

1. [Setup Development Environment](#setup-development-environment)
2. [Design Robot Architecture](#design-robot-architecture)
3. [Implement Nodes](#implement-nodes)
   - [LLM Node](#llm-node)
   - [TTS Node](#tts-node)
   - [Control Node](#control-node)
   - [Speech Recognition Node (Optional)](#speech-recognition-node-optional)
4. [Build and Run](#build-and-run)
5. [Test and Iterate](#test-and-iterate)
6. [Add Personalization](#add-personalization)
7. [Deploy on Hardware](#deploy-on-hardware)

---

## Setup Development Environment

### 1. Install Ubuntu (Recommended: 22.04 LTS)
- Ubuntu 22.04 LTS or any other compatible version is recommended 
  to ensure compatibility with ROS2. It can be installed directly or using a virtual machine.

### 2. Install ROS2 (Humble or Rolling)
- ROS2 should be installed by following the official 
[ROS2 Installation Guide](https://docs.ros.org/en/rolling/Installation.html). 
It is recommended to choose either **Humble** (for long-term support) or **Rolling** (for the latest features).

### 3. Install Dependencies
- Python, pip, and necessary libraries should be installed 
  for integration with the LLM and TTS systems. The following 
  commands are used to install dependencies:

```bash
sudo apt install python3-pip
pip install transformers openai gTTS
```

### 4. Set Up the Workspace
- A ROS2 workspace should be created for organizing the project 
  and compiling the code. The following commands set up and build the workspace:

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
colcon build
```

---

## Design Robot Architecture

### Nodes and Topics
- **LLM Node**: This node processes user input and generates appropriate responses using a large language model (LLM) like GPT-3.
- **TTS Node**: This node converts the LLM output (text) into speech using a TTS library like `gTTS`.
- **Speech Recognition Node (Optional)**: This node is optional and processes speech input, converting it into text using libraries like `speech_recognition`.
- **Control Node**: The control node coordinates the communication between other nodes and controls the flow of data.

### Interfaces (Topics and Services)
- Example topics:
  - `/user_input` (String): For user questions.
  - `/llm_output` (String): For LLM-generated responses.
  - `/tts_output` (Audio): For TTS-generated speech.

---

## Implement Nodes

### LLM Node
- The LLM node listens for user input, processes it using an LLM, 
  and publishes the response. The following code implements the LLM node 
  using the Hugging Face `transformers` library:

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String
from transformers import pipeline

class LLMNode(Node):
    def __init__(self):
        super().__init__('llm_node')
        self.subscriber = self.create_subscription(String, '/user_input', self.process_input, 10)
        self.publisher = self.create_publisher(String, '/llm_output', 10)
        self.llm = pipeline('text-generation', model='gpt-3.5-turbo')  # Model selection

    def process_input(self, msg):
        user_input = msg.data
        response = self.llm(user_input, max_length=100)[0]['generated_text']
        self.publisher.publish(String(data=response))

def main(args=None):
    rclpy.init(args=args)
    node = LLMNode()
    rclpy.spin(node)
    rclpy.shutdown()
```

### TTS Node
- The TTS node listens for the LLM's output and converts it into speech. 
  The following code uses the `gTTS` library to generate speech from text:

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String
from gtts import gTTS
import os

class TTSNode(Node):
    def __init__(self):
        super().__init__('tts_node')
        self.subscriber = self.create_subscription(String, '/llm_output', self.speak, 10)

    def speak(self, msg):
        tts = gTTS(text=msg.data, lang='en')
        tts.save("output.mp3")
        os.system("mpg123 output.mp3")  # Play audio

def main(args=None):
    rclpy.init(args=args)
    node = TTSNode()
    rclpy.spin(node)
    rclpy.shutdown()
```

### Control Node
- The Control Node coordinates the interaction between the user input, LLM, and TTS nodes, 
  ensuring that the appropriate steps are taken in the right sequence. 
  It can be implemented simply by subscribing to and publishing messages between the nodes.

### Speech Recognition Node (Optional)
- If voice interaction is required, a speech recognition node can be added. 
  This node listens to user speech, converts it into text, 
  and publishes it to the `/user_input` topic. Libraries like `speech_recognition` can be used for this functionality.

---

## Build and Run

### 1. Build Your ROS2 Packages
- Each node should be placed in its respective package under the `~/ros2_ws/src` directory. 
  The workspace should be built with the following command:

```bash
cd ~/ros2_ws
colcon build
```

### 2. Launch the System
- A launch file should be created to launch all the necessary nodes simultaneously. 
  An example launch file (`robot_launch.py`) is provided below:

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(package='llm_package', executable='llm_node', name='llm_node'),
        Node(package='tts_package', executable='tts_node', name='tts_node'),
        # Optionally add Speech Recognition node
        # Node(package='speech_recognition_package', executable='speech_recognition_node', name='speech_recognition_node')
    ])
```

- To run the launch file, the following command is used:

```bash
ros2 launch your_package robot_launch.py
```

---

## Test and Iterate

- To test the system, the user input should be published to the `/user_input` topic. 
  The LLM node processes this input and sends the response to the TTS node, which will then generate speech.

Example command to publish test input:

```bash
ros2 topic pub /user_input std_msgs/String "data: 'How are you today?'"
```

---

## Add Personalization

- Personalization features, such as tracking vocabulary level, grammar proficiency, or user progress, can be added. 
  Data can be stored in a persistent database (e.g., SQLite or JSON). T
  he system can offer quizzes or conversational scenarios based on the user’s level.

---

## Deploy on Hardware

### 1. Hardware Setup
- The system can be deployed on a Raspberry Pi or another suitable platform, 
  and sensors and actuators can be added to interact with the environment.

### 2. Sensor and Actuator Integration
- If required, sensors such as cameras or distance sensors, 
  and actuators for physical movement or interaction, can be integrated into the system.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
