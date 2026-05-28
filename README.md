ROS 2 Pub/Sub Demo — Modular Communication in Robotics & Cyber-Physical Systems
A minimal but complete ROS 2 publisher–subscriber example built with rclpy, demonstrating the decoupled, topic-based communication that underpins modular robotics and cyber-physical systems.

Why This Pattern Matters
In any real robotics system — a quadruped, an autonomous vehicle, or an industrial CPS — components must share data without being tightly coupled to each other. ROS 2 solves this with a publish/subscribe model:

A publisher broadcasts data on a named topic
Any number of subscribers listen on that topic independently
Neither side knows or cares about the other's implementation

This is the same pattern used in:

Sensor pipelines (LiDAR → perception node → planner)
Motor control loops (controller → joint state publisher → hardware interface)
Industrial automation (PLC data → SCADA → anomaly detector)


Architecture
┌─────────────────────┐          /chatter          ┌──────────────────────┐
│    MyPublisher      │  ──────────────────────►   │    MySubscriber      │
│  (my_publisher)     │   std_msgs/msg/String       │  (my_subscriber)     │
│                     │   @ 1 Hz                    │                      │
│  publishes:         │                             │  logs received msg   │
│  "Hello Subscriber" │                             │                      │
└─────────────────────┘                             └──────────────────────┘
ComponentNode NameTopicMessage TypeRatepublisher.pymy_publisher/chatterstd_msgs/String1 Hzsubscriber.pymy_subscriber/chatterstd_msgs/Stringevent-driven

Create From Scratch
Follow these steps to recreate this package entirely from scratch.
1. Create the workspace
bashmkdir -p ~/ros2_ws/src
cd ~/ros2_ws
2. Create the ROS 2 Python package
bashcd src
ros2 pkg create --build-type ament_python pubsub
This generates the full package skeleton:
pubsub/
├── pubsub/
│   └── __init__.py       ← Python module folder (your nodes go here)
├── package.xml
├── setup.cfg
├── setup.py
└── resource/
    └── pubsub
3. Write the nodes
Create src/pubsub/pubsub/publisher.py:
pythonimport rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MyPublisher(Node):
    def __init__(self):
        super().__init__('my_publisher')
        self.publisher = self.create_publisher(String, 'chatter', 10)
        self.timer = self.create_timer(1.0, self.publish_message)

    def publish_message(self):
        msg = String()
        msg.data = "Hello Subscriber"
        self.publisher.publish(msg)
        self.get_logger().info(f'publishing: {msg.data}')

def main(args=None):
    rclpy.init(args=args)
    node = MyPublisher()
    rclpy.spin(node)
    rclpy.shutdown()

if __name__ == '__main__':
    main()
Create src/pubsub/pubsub/subscriber.py:
pythonimport rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MySubscriber(Node):
    def __init__(self):
        super().__init__('my_subscriber')
        self.subscription = self.create_subscription(
            String, 'chatter', self.listener_callback, 10)

    def listener_callback(self, msg):
        self.get_logger().info(f'Received : {msg.data}')

def main(args=None):
    rclpy.init(args=args)
    node = MySubscriber()
    rclpy.spin(node)
    rclpy.shutdown()

if __name__ == '__main__':
    main()
4. Register the entry points
Edit setup.py and add the two scripts under console_scripts:
pythonentry_points={
    'console_scripts': [
        'pub = pubsub.publisher:main',
        'sub = pubsub.subscriber:main',
    ],
},
5. Build the workspace
bashcd ~/ros2_ws
colcon build
6. Source the workspace
bashsource install/setup.bash

Tip: Add this to your ~/.bashrc so you don't have to run it every time:
bashecho "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc

7. Run the nodes
Open two terminals. Source the workspace in each.
Terminal 1 — Publisher:
bashsource ~/ros2_ws/install/setup.bash
ros2 run pubsub pub
Terminal 2 — Subscriber:
bashsource ~/ros2_ws/install/setup.bash
ros2 run pubsub sub

Output
Publisher:
Show Image
Subscriber:
Show Image

Useful ROS 2 Commands
bash# List all active topics
ros2 topic list

# Inspect messages on /chatter in real time
ros2 topic echo /chatter

# Check publish rate
ros2 topic hz /chatter

# See node info
ros2 node info /my_publisher
ros2 node info /my_subscriber

# View the full node graph
rqt_graph

Project Structure
ros2_ws/
└── src/
    └── pubsub/
        ├── pubsub/
        │   ├── __init__.py
        │   ├── publisher.py
        │   └── subscriber.py
        ├── package.xml
        ├── setup.cfg
        └── setup.py

Extending This
Swap out String for any ROS 2 message type to model real robotics data:
Use CaseMessage TypeTopic ExampleIMU datasensor_msgs/Imu/imu/dataJoint statessensor_msgs/JointState/joint_statesCamera feedsensor_msgs/Image/camera/image_rawRobot velocitygeometry_msgs/Twist/cmd_velCustom servo anglesstd_msgs/Float64MultiArray/servo_angles

Related Work
This pub/sub foundation is used in my ongoing 12-servo quadruped robot project, where individual nodes handle:

Gait planning → publishes target joint angles
Servo controller → subscribes and drives hardware
Perception layer → subscribes to camera, publishes object detections


Requirements

Ubuntu 22.04 / 24.04
ROS 2 Jazzy (or Humble)
Python 3.10+


Author
Anup — MSc Computer Science (Industry 4.0, Robotics & Automation), HAM Munich
GitHub · LinkedIn

License
MIT
