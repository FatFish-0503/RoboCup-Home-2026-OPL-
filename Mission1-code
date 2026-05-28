#!/usr/bin/env python3.8
import rospy
from sensor_msgs.msg import Image, Imu
from nav_msgs.msg import Odometry
from cv_bridge import CvBridge
from std_msgs.msg import String
from geometry_msgs.msg import Twist
from mr_voice.msg import Voice
from ultralytics import YOLO
import cv2
from RobotChassis import RobotChassis # Your chassis driver
from tf.transformations import euler_from_quaternion
from open_manipulator_msgs.srv import SetJointPosition, SetJointPositionRequest
import subprocess
import sounddevice as sd
from scipy.io.wavfile import write
import requests
import json
import base64
import time
import re
import numpy as np
import math
import matplotlib.pyplot as plt
import time
import pyautogui
import os
import tempfile
import subprocess
import librosa

OPENROUTER_API_KEY = "YOUR_API_KEY"
VISION_MODEL = "qwen/qwen3.5-flash-02-23"
MODEL = "google/gemini-2.5-flash-lite"
CHAIR_MODEL = "qwen/qwen3.5-flash-02-23"
AUDIO_DEVICE = "default" 
AUDIO_FS = 16000
AUDIO_CHANNELS = 1     
API_KEY = "YOUR_API_KEY"                                                                                     

_image = None
_depth = None
_image_seq = 0
_depth_seq = 0
_voice = None
_odom = None
_low_image = None
_low_depth = None
CvBr = CvBridge()
code = 0 
_current_cam = 1 
_sub_cam_image = None
_sub_cam_depth = None
bridge = CvBridge()
_audio_path = None
_new_audio_flag = False

NAME_LIST = [
     "sophie", "kevin", "julia", "gabrielle", "emma", "robin",
    "noah", "sara", "john", "harrie", "laura", "liam",
    "peter", "lucas", "susan", "william"
]


DRINK_LIST= [
 "lemonade", "coffee can", "fanta", "yakult",
    "green tea", "milk", "red bull can"
]

color_ranges = {
 'red': [(0, 100, 100), (10, 255, 255), (160, 100, 100), (180, 255, 255)],
 'orange': [(11, 100, 100), (25, 255, 255)],
 'yellow': [(26, 100, 100), (35, 255, 255)],
 'green': [(36, 100, 100), (85, 255, 255)],
 'cyan': [(86, 100, 100), (95, 255, 255)],
 'blue': [(96, 100, 100), (130, 255, 255)],
 'violet': [(131, 100, 100), (145, 255, 255)],
 'magenta': [(146, 100, 100), (159, 255, 255)],
 'black': [(0, 0, 0), (180, 255, 50)],
 'white': [(0, 0, 200), (180, 30, 255)]
}
model = YOLO("yolov8n-pose.pt")
# def tezhengyi():

def callback_voice(msg):
    global _voice
    _voice = msg

def callback_odom(msg):
    global _odom
    _odom = msg

def callback_path(msg):
    global _audio_path, _new_audio_flag
    _audio_path = msg.data
    _new_audio_flag = True
    print("new audio path =", _audio_path)

def callback_image(msg):
    global _image, _image_seq
    try:
        _image = bridge.imgmsg_to_cv2(msg, "bgr8")
        _image_seq += 1
    except Exception as e:
        print("callback_image error:", e)


def callback_depth(msg):
    global _depth, _depth_seq
    try:
        _depth = bridge.imgmsg_to_cv2(msg, "passthrough")
        _depth_seq += 1
    except Exception as e:
        print("callback_depth error:", e)

def callback_low_image(msg):
    global _low_image
    _low_image = CvBridge().imgmsg_to_cv2(msg, "bgr8")

def callback_low_depth(msg):
    global _low_depth
    _low_depth = CvBridge().imgmsg_to_cv2(msg)

def callback_Imu(msg):
    global _Imu
    _Imu = msg

def move(dis, turn):
    fb = Twist()
    fb.linear.x = dis
    fb.angular.z = turn
    pub_cmd.publish(fb)

def say(text):
    os.system(f"espeak -s 120 \"{text}\"")
    rospy.sleep(2)

def get_pose_box(user, image_shape, padding=20):
    if user is None or len(user) == 0:
        return None

    h, w = image_shape[:2]
    xs = []
    ys = []

    for p in user:
        x, y = map(int, p)
        if 0 <= x < w and 0 <= y < h:
            xs.append(x)
            ys.append(y)

    if len(xs) == 0 or len(ys) == 0:
        return None

    x1 = max(0, min(xs) - padding)
    y1 = max(0, min(ys) - padding)
    x2 = min(w - 1, max(xs) + padding)
    y2 = min(h - 1, max(ys) + padding)

    return x1, y1, x2, y2

def wait_for_new_audio(timeout=5):
    global _audio_path, _new_audio_flag, _voice

    start = time.time()

    while time.time() - start < timeout:
        if _voice is not None:
            path = _audio_path
            _voice = None
            if os.path.exists(path):
                _new_audio_flag = False
                print("use audio file:", path)
                return path

        rospy.sleep(0.05)

    print("no new audio")
    return None

AUDIO_DEVICE = "plughw:1,0"  

def encode_audio(audio_path):
    with open(audio_path, "rb") as f:
        return base64.b64encode(f.read()).decode("utf-8")

def detect_bell_from_file(audio_path):
    try:
        target_path = "/home/robot/doorbell/dragon-studio-doorbell-ding-dong-482879.wav"

        SAMPLE_RATE = 44100
        THRESHOLD = 145000

        target_y, target_sr = librosa.load(target_path, sr=SAMPLE_RATE)
        audio_y, audio_sr = librosa.load(audio_path, sr=SAMPLE_RATE)

        target_mfcc = librosa.feature.mfcc(y=target_y, sr=SAMPLE_RATE, n_mfcc=40).T
        audio_mfcc = librosa.feature.mfcc(y=audio_y, sr=SAMPLE_RATE, n_mfcc=40).T

        n = min(len(target_mfcc), len(audio_mfcc))
        target_mfcc = target_mfcc[-n:]
        audio_mfcc = audio_mfcc[-n:]

        distance = np.sum(((audio_mfcc[:, 1:] - target_mfcc[:, 1:]) ** 2) ** 0.5)

        print("[BELL] distance =", distance)

        if distance < THRESHOLD:
            return True
        else:
            return False

    except Exception as e:
        print("[BELL] detect error:", e)
        return False
    
def match_from_list(text, valid_list):
    if text is None:
        return None

    text = text.lower().strip()

    for item in valid_list:
        if item in text:
            return item

    return None

def wait_new_frame(old_seq, timeout=3.0):
    start = rospy.get_time()
    while rospy.get_time() - start < timeout:
        if _image is not None and _image_seq > old_seq:
            return True
        rospy.sleep(0.05)
    return False

def save_user_crop(image, user, save_path=None):
    box = get_pose_box(user, image.shape)
    if box is None:
        return None

    x1, y1, x2, y2 = box
    crop = image[y1:y2, x1:x2]

    if crop.size == 0:
        return None

    if save_path is None:
        fd, save_path = tempfile.mkstemp(suffix=".jpg")
        os.close(fd)

    cv2.imwrite(save_path, crop)
    return save_path

def bag_any_xy(image):
    h, w, c = image.shape
    result = luggage_model(image, verbose=False)[0]
    boxes = result.boxes

    best_box = None
    best_score = -1

    for cls, xyxy in zip(boxes.cls, boxes.xyxy):
        x1, y1, x2, y2 = map(int, xyxy)

        cx = (x1 + x2) // 2
        cy = (y1 + y2) // 2
        area = (x2 - x1) * (y2 - y1)

        if cy > h * 0.9:
            continue

        score = area - abs(cx - w // 2) * 150

        if score > best_score:
            best_score = score
            best_box = (x1, y1, x2, y2)

    if best_box is None:
        return 0, 0, None

    x1, y1, x2, y2 = best_box
    cx = (x1 + x2) // 2
    cy = (y1 + y2) // 2

    cv2.rectangle(image, (x1, y1), (x2, y2), (255, 0, 0), 2)
    return cx, cy, (x1, y1, x2, y2)

def is_bag_handover_from_guest2(image, depth, user):
    h, w = depth.shape

    person_box = get_pose_box(user, image.shape)
    if person_box is None:
        return False, 0, 0, 0

    px1, py1, px2, py2 = person_box
    pcx = (px1 + px2) // 2
    pcy = (py1 + py2) // 2
    pw = px2 - px1
    ph = py2 - py1

    cv2.rectangle(image, (px1, py1), (px2, py2), (0, 255, 255), 2)

    bagx, bagy, bag_box = bag_any_xy(image)
    if bag_box is None:
        return False, 0, 0, 0

    bx1, by1, bx2, by2 = bag_box
    bcx = (bx1 + bx2) // 2
    bcy = (by1 + by2) // 2

    if bagx < 0 or bagx >= w or bagy < 0 or bagy >= h:
        return False, 0, 0, 0

    bag_depth = depth[bagy][bagx]

    if bag_depth == 0:
        return False, bagx, bagy, 0

    if abs(bcx - pcx) > pw * 1.0:
        return False, bagx, bagy, bag_depth

    if bcy > py2 + ph * 0.15:
        return False, bagx, bagy, bag_depth

    if bcy < py1 + ph * 0.10:
        return False, bagx, bagy, bag_depth

    if bag_depth > 1200:
        return False, bagx, bagy, bag_depth

    if bag_depth < 250:
        return False, bagx, bagy, bag_depth

    return True, bagx, bagy, bag_depth

def crop_middle_robot(image, depth):
    h, w = image.shape[:2]

    # whole middle area
    x1 = int(w * 0.20)
    x2 = int(w * 0.80)

    y1 = int(h * 0.10)
    y2 = int(h * 0.95)

    middle_img = image[y1:y2, x1:x2]
    middle_depth = depth[y1:y2, x1:x2]

    return middle_img, middle_depth

def find_user(img, depth):    
    global model, image

    h, w = depth.shape
    keypoints = model(image, show=True, save=True)[0].keypoints
    user = []
    user_d = -1
    for pose in keypoints.xy:
        mid_d = -1
        for p in pose:
            x, y = map(int , p)
            cv2.circle(image, (x, y), 3, (0, 255, 0), -1)
            if x >= w or y >= h:
                continue
            if depth[y][x] > 0:
                if mid_d == -1 or depth[y][x] < mid_d:
                    mid_d = depth[y][x]
        if (user_d == -1 or mid_d < user_d) and mid_d != -1:
            user_d = mid_d
            user = pose
    return user


def user_point_to_LR(pose):
    if pose is None: return 0
    if len(pose) == 0: return 0
    x0, y0 = map(int, pose[0])
    x8, y8 = map(int, pose[10])
    if x0 == x8: return 0
    k = (y0 - y8) / (x0 - x8)
    if abs(k) > 1.2: return 0
    if k > 0: return 1
    return -1


def bag_xy(dir,image):
    h, w, cc = image.shape
    result = luggage_model(image, verbose=False)[0]
    boxes = result.boxes
   
    for c, xyxy in zip(boxes.cls, boxes.xyxy):
        x1, y1, x2, y2 = map(int, xyxy)
        cv2.rectangle(image, (x1, y1), (x2, y2), (0, 255, 0), 2)
        b = 0
        if int((x1+x2)/2) > w//2:
            b = 1
        if dir == b:
            cv2.rectangle(image, (x1, y1), (x2, y2), (255, 0, 0), 2)
            return int((x1+x2)/2), int((y1+y2)/2)
    return 0,0

def change_to_cam(idx):
    global _sub_cam_image, _sub_cam_depth, _current_cam
    _current_cam = idx

    if _sub_cam_image is not None:
        _sub_cam_image.unregister()
    if _sub_cam_depth is not None:
        _sub_cam_depth.unregister()

    if idx == 1:
        _sub_cam_image = rospy.Subscriber("/cam2/color/image_raw", Image, callback_image)
        _sub_cam_depth = rospy.Subscriber("/cam2/depth/image_raw", Image, callback_depth)
    elif idx == 2:
        _sub_cam_image = rospy.Subscriber("/cam1/color/image_raw", Image, callback_image)
        _sub_cam_depth = rospy.Subscriber("/cam1/depth/image_raw", Image, callback_depth)

def ai_vision_request(mode, prompt, api_key=OPENROUTER_API_KEY, image_path=None, model=VISION_MODEL):
    url = "https://openrouter.ai/api/v1/chat/completions"

    if mode == 1 and image_path:
        with open(image_path, "rb") as image_file:
            base64_image = base64.b64encode(image_file.read()).decode("utf-8")

        content = [
            {"type": "text", "text": prompt},
            {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{base64_image}"}}
        ]
    else:
        content = prompt

    payload = {
        "model": model,
        "messages": [{"role": "user", "content": content}],
        "temperature": 0
    }

    start_time = time.time()

    try:
        response = requests.post(
            url=url,
            headers={
                "Authorization": f"Bearer {api_key}",
                "Content-Type": "application/json",
            },
            data=json.dumps(payload),
            timeout=30
        )
        print(response)

        execution_time = time.time() - start_time
        result = response.json()

        if "choices" in result:
            answer = result["choices"][0]["message"]["content"]
            
            #remove json
            answer = re.sub(r"^```json\s*", "", answer.strip())
            answer = re.sub(r"^```\s*", "", answer.strip())
            answer = re.sub(r"\s*```$", "", answer.strip())

            return {
                "success": True,
                "answer": answer.strip(),
                "time": f"{execution_time:.2f}s"
            }
        else:
            return {"success": False, "error": result}

    except Exception as e:
        return {"success": False, "error": str(e)}

def draw_chair_numbers(image):
    check_empty_img = image.copy()

    font = cv2.FONT_HERSHEY_SIMPLEX
    font_scale = 2.0
    font_color = (255, 255, 255)
    font_thickness = 5

    h, w = check_empty_img.shape[:2]

    print("image size =", w, h)

    # 🔥 用比例，而不是寫死 640, 320
    positions = {
        1: (int(w * 0.15), int(h * 0.65)),  # 左椅
        2: (int(w * 0.50), int(h * 0.70)),  # 中間椅
        3: (int(w * 0.80), int(h * 0.65)),  # 右椅
    }

    for number, pos in positions.items():
        text = str(number)
        text_size = cv2.getTextSize(text, font, font_scale, font_thickness)[0]

        x, y = pos

        # 黑底
        rectangle_start = (x - 15, y + 15)
        rectangle_end = (x + text_size[0] + 15, y - text_size[1] - 15)
        cv2.rectangle(check_empty_img, rectangle_start, rectangle_end, (0, 0, 0), cv2.FILLED)

        # 數字
        cv2.putText(
            check_empty_img,
            text,
            pos,
            font,
            font_scale,
            font_color,
            font_thickness,
            cv2.LINE_AA
        )

        # debug點
        cv2.circle(check_empty_img, pos, 5, (0, 255, 0), -1)

    return check_empty_img

def chair_id_to_seat(chair_id):
    if chair_id == 1:
        return "left"
    elif chair_id == 2:
        return "middle"
    elif chair_id == 3:
        return "right"
    return None

def audio_extract_name(audio_path, api_key, model=MODEL):
    try:
        base64_audio = encode_audio(audio_path)

        prompt = """
Transcribe exactly what the speaker says.

Rules:
- Do NOT guess.
- Do NOT invent names.
- If unclear, output exactly: unclear
- Output only the spoken sentence.

NAME_LIST = [
     "sophie", "kevin", "julia", "gabrielle", "emma", "robin",
    "noah", "sara", "john", "harrie", "laura", "liam",
    "peter", "lucas", "susan", "william"
]


DRINK_LIST= [
 "lemonade", "coffee can", "fanta", "yakult",
    "green tea", "milk", "red bull can"
]

These are the name list and drink list. When you are replying a name, make sure it is in this list. If not, find a closest name in this list comparing with the audio.
This also occurs to the drink.
"""

        payload = {
            "model": model,
            "messages": [{
                "role": "user",
                "content": [
                    {"type": "text", "text": prompt},
                    {
                        "type": "input_audio",
                        "input_audio": {
                            "data": base64_audio,
                            "format": "wav"
                        }
                    }
                ]
            }],
            "temperature": 0
        }

        response = requests.post(
            url="https://openrouter.ai/api/v1/chat/completions",
            headers={
                "Authorization": f"Bearer {api_key}",
                "Content-Type": "application/json",
            },
            data=json.dumps(payload),
            timeout=15
        )

        result = response.json()

        if "choices" not in result:
            return {"success": False, "name": None}

        text = result["choices"][0]["message"].get("content")
        if text is None:
            return {"success": False, "name": None}

        text = text.strip().lower()
        print("name transcript =", text)

        name = match_from_list(text, NAME_LIST)

        if name is None:
            return {"success": False, "name": None}

        return {"success": True, "name": name}

    except Exception as e:
        print("audio_extract_name exception:", e)
        return {"success": False, "name": None}
    
def extract_name(text):
    text = text.lower().strip()

    patterns = ["my name is", "i am", "i'm"]

    for p in patterns:
        if p in text:
            name = text.split(p)[-1].strip()
            if name:
                return name.split()[0]

    words = text.split()
    if len(words) == 1:
        return words[0]

    return None

def get_name_with_retry(default_name="friend", max_retry=2):
    for attempt in range(max_retry + 1):
        print(f"[Name] attempt {attempt+1}/{max_retry+1}")

        wav_path = record_audio_arecord(f"/tmp/guest_name_{attempt}.wav", 5)
        if wav_path is None:
            continue

        res = audio_extract_name(wav_path, OPENROUTER_API_KEY)
        print("name result =", res)

        if res["success"] and res["name"] is not None:
            return str(res["name"]).strip()

    return default_name


def get_drink_with_retry(default_drink="water", max_retry=2):
    for attempt in range(max_retry + 1):
        print(f"[Drink] attempt {attempt+1}/{max_retry+1}")

        wav_path = record_audio_arecord(f"/tmp/guest_drink_{attempt}.wav", 5)
        if wav_path is None:
            continue

        res = audio_extract_drink(wav_path, OPENROUTER_API_KEY)
        print("drink result =", res)

        if res["success"] and res["drink"] is not None:
            return str(res["drink"]).strip()

    return default_drink

def audio_extract_drink(audio_path, api_key, model=MODEL):
    try:
        base64_audio = encode_audio(audio_path)

        prompt = """
Transcribe exactly what the speaker says.

Rules:
- Do NOT guess.
- Do NOT invent drinks.
- If unclear, output exactly: unclear
- Output only the spoken sentence.
"""

        payload = {
            "model": model,
            "messages": [{
                "role": "user",
                "content": [
                    {"type": "text", "text": prompt},
                    {
                        "type": "input_audio",
                        "input_audio": {
                            "data": base64_audio,
                            "format": "wav"
                        }
                    }
                ]
            }],
            "temperature": 0
        }

        response = requests.post(
            url="https://openrouter.ai/api/v1/chat/completions",
            headers={
                "Authorization": f"Bearer {api_key}",
                "Content-Type": "application/json",
            },
            data=json.dumps(payload),
            timeout=15
        )

        result = response.json()

        if "choices" not in result:
            return {"success": False, "drink": None}

        text = result["choices"][0]["message"].get("content")
        if text is None:
            return {"success": False, "drink": None}

        text = text.strip().lower()
        print("drink transcript =", text)

        drink = match_from_list(text, DRINK_LIST)

        if drink is None:
            return {"success": False, "drink": None}

        return {"success": True, "drink": drink}

    except Exception as e:
        print("audio_extract_drink exception:", e)
        return {"success": False, "drink": None}
    
def get_name_with_retry(default_name="friend", max_retry=2):
    for attempt in range(max_retry + 1):
        print(f"[Name] attempt {attempt+1}/{max_retry+1}")

        wav_path = record_audio(filename=f"/tmp/guest_name_{attempt}.wav", seconds=3)
        if wav_path is None:
            print("record failed")
            continue

        res = audio_extract_name(wav_path)
        print("name result =", res)

        if res["success"] and res["name"] is not None:
            name = str(res["name"]).strip()
            if name != "":
                return name

    print("fallback name =", default_name)
    return default_name

# ===== odom turn =====
def get_yaw_from_odom():
    global _odom
    q = [
        _odom.pose.pose.orientation.x,
        _odom.pose.pose.orientation.y,
        _odom.pose.pose.orientation.z,
        _odom.pose.pose.orientation.w
    ]
    _, _, yaw = euler_from_quaternion(q)
    return yaw

def normalize_angle(angle):
    while angle > np.pi:
        angle -= 2*np.pi
    while angle < -np.pi:
        angle += 2*np.pi
    return angle

def turn_to(angle, speed):
    max_speed = 1.82
    start = rospy.get_time()

    while True:
        yaw = get_yaw_from_odom()
        e = normalize_angle(angle - yaw)

        if abs(e) < 0.01 or rospy.get_time()-start > 8:
            break

        move(0.0, max_speed*speed*e)
        rospy.Rate(20).sleep()

    move(0,0)

def turn(angle):
    yaw = get_yaw_from_odom()
    target = normalize_angle(yaw + angle)
    turn_to(target, 0.5)

def detect_real_xyz(x1,y1,d):
    global image
    h, w, c = image.shape
    y1 -= h/2
    x1 -= w/2
    y = y1/h*2*d*math.tan(45.5*3.14159/360)
    x = x1/w*2*d*math.tan(58.4*3.14159/360)
    return x,y,d 

def find_human_xy(user,image):
    x1,y1,x2,y2 = 0,0,0,0
    num = 0
    if user is not None:
        for pp in user:
            x, y = map(int, pp)
            if num == 11: x1,y1 = x,y
            elif num == 12: x2,y2 = x,y
            num+=1

    if (x1 or x2) and (y1 or y2):
        x3 = int((x1+x2)/2) if x1 and x2 else (x1 or x2)
        y3 = int((y1+y2)/2) if y1 and y2 else (y1 or y2)
        return x3,y3
    return 0,0


def find_speed(depth,x3,y3,lsp):
    dis = depth[y3][x3]
    v = dis-650
    speed = float(v*0.0005)
    if speed > 0.3: speed = 0.3
    if speed-lsp>0.01: speed = lsp+0.01
    if lsp-speed>0.01: speed = lsp-0.01
    return speed

def set_joints(j1,j2,j3,j4,t):
    rospy.wait_for_service("/goal_joint_space_path")
    try:
        s = rospy.ServiceProxy("/goal_joint_space_path", SetJointPosition)
        req = SetJointPositionRequest()
        req.joint_position.joint_name = ["joint1","joint2","joint3","joint4"]
        req.joint_position.position = [j1,j2,j3,j4]
        req.path_time = t
        return s(req)
    except:
        return False

def extract_after_phrase(data, phrase, options):
    data = data.lower()
    if phrase in data:
        words = data.split()
        phrase_words = phrase.split()
        for i in range(len(words) - len(phrase_words)):
            if words[i:i+len(phrase_words)] == phrase_words:
                for j in range(1, 4):
                    if i + len(phrase_words) + j <= len(words):
                        guess = ' '.join(words[i + len(phrase_words): i + len(phrase_words) + j])
                        if not options or guess in options:
                            return guess
    return None

def extract_name_after_phrase(data, phrase):
    return extract_after_phrase(data, phrase, nameslist)

def extract_drink(data, drinklist):
    d = extract_after_phrase(data, "i want", drinklist)
    if not d:
        d = extract_after_phrase(data, "i would like", drinklist)
    if not d:
        d = extract_after_phrase(data, "give me", drinklist)
    return d

def detect_people_positions(image, model):
    results = model(image, verbose=False)[0]
    people = []
    for box, cls, conf in zip(results.boxes.xyxy, results.boxes.cls, results.boxes.conf):
        if int(cls)==0 and conf>0.6:
            x1,y1,x2,y2 = map(int,box)
            people.append(((x1+x2)//2,(y1+y2)//2))
    return people

def draw_chair_numbers(image):
    img = image.copy()

    font = cv2.FONT_HERSHEY_SIMPLEX
    font_scale = 2.0
    font_color = (255, 255, 255)
    font_thickness = 5

    h, w = img.shape[:2]

    positions = {
        1: (int(w * 0.15), int(h * 0.65)),  # 左
        2: (int(w * 0.50), int(h * 0.70)),  # 中
        3: (int(w * 0.80), int(h * 0.65)),  # 右
    }

    for number, (x, y) in positions.items():
        text = str(number)
        text_size = cv2.getTextSize(text, font, font_scale, font_thickness)[0]

        cv2.rectangle(img, (x-15, y+15), (x+text_size[0]+15, y-text_size[1]-15), (0,0,0), -1)
        cv2.putText(img, text, (x,y), font, font_scale, font_color, font_thickness, cv2.LINE_AA)

    return img


def encode_image(path):
    with open(path, "rb") as f:
        return base64.b64encode(f.read()).decode("utf-8")


def follow():
    global _voice

    say("I will follow you now. Say stop to stop me.")
    print("[FOLLOW] start")

    desired_dist_mm = 700
    max_forward_speed = 0.28
    max_backward_speed = -0.12
    max_turn_speed = 0.45

    lost_count = 0
    max_lost_count = 30
    last_linear = 0.0

    _voice = None  # 清走舊語音

    while not rospy.is_shutdown():

        # ===== mr_voice stop check =====
        if _voice is not None:
            text = _voice.text.strip().lower()
            print("[FOLLOW] voice =", text)

            _voice = None

            if "stop" in text:
                say("stop")
                move(0, 0)
                return

        image = _image.copy()
        depth = _depth.copy()

        if image is None or depth is None:
            rospy.sleep(0.05)
            continue

        h, w = image.shape[:2]
        center_x = w // 2

        user = find_user(image, depth)

        if user is None or len(user) == 0:
            lost_count += 1
            move(0.0, 0.18)

            if lost_count > max_lost_count:
                say("I lost you")
                move(0, 0)
                return

            rospy.sleep(0.05)
            continue

        lost_count = 0

        x, y = find_human_xy(user, image)

        if x == 0 and y == 0:
            move(0, 0)
            rospy.sleep(0.05)
            continue

        if not (0 <= y < depth.shape[0] and 0 <= x < depth.shape[1]):
            move(0, 0)
            rospy.sleep(0.05)
            continue

        d = depth[y][x]

        if d <= 0:
            move(0, 0)
            rospy.sleep(0.05)
            continue

        error_x = x - center_x
        turn_speed = -error_x * 0.002
        turn_speed = max(-max_turn_speed, min(max_turn_speed, turn_speed))

        error_d = d - desired_dist_mm
        linear_speed = error_d * 0.00045
        linear_speed = max(max_backward_speed, min(max_forward_speed, linear_speed))

        if linear_speed - last_linear > 0.02:
            linear_speed = last_linear + 0.02
        if last_linear - linear_speed > 0.02:
            linear_speed = last_linear - 0.02

        last_linear = linear_speed

        print("[FOLLOW] linear =", linear_speed, "turn =", turn_speed)

        move(linear_speed, turn_speed)
        rospy.sleep(0.05)

def ask_empty_chair_api(image_path):
    base64_image = encode_image(image_path)

    prompt = """
Look at the image.
There are three chairs labeled 1, 2, and 3.
Tell me which chair is empty.

Return JSON only:
{"empty_chair": 1}
"""

    payload = {
        "model": CHAIR_MODEL,
        "messages": [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": prompt},
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": f"data:image/jpeg;base64,{base64_image}"
                        }
                    }
                ]
            }
        ],
        "temperature": 0
    }

    try:
        response = requests.post(
            "https://openrouter.ai/api/v1/chat/completions",
            headers={
                "Authorization": f"Bearer {OPENROUTER_API_KEY}",
                "Content-Type": "application/json"
            },
            data=json.dumps(payload),
            timeout=20
        )

        result = response.json()

        if "choices" not in result:
            print("API error:", result)
            return None

        answer = result["choices"][0]["message"]["content"].strip()

        # clean ```json```
        answer = re.sub(r"^```json\s*", "", answer)
        answer = re.sub(r"^```\s*", "", answer)
        answer = re.sub(r"\s*```$", "", answer)

        data = json.loads(answer)
        return data.get("empty_chair")

    except Exception as e:
        print("API fail:", e)
        return None


def detect_empty_seat(image):
    # 存暫存圖
    path = "/tmp/chair.jpg"
    labeled = draw_chair_numbers(image)
    cv2.imwrite(path, labeled)

    results = []
    for i in range(1):
        cid = ask_empty_chair_api(path)
        print(f"trial {i+1} =", cid)
        if cid in [1,2,3]:
            results.append(cid)

    if len(results) == 0:
        return None

    cid = max(set(results), key=results.count)

    if cid == 1:
        return "left"
    elif cid == 2:
        return "middle"
    elif cid == 3:
        return "right"

def turn_back(speed=0.2):
    yaw = get_yaw_from_odom()
    target = normalize_angle(yaw + np.pi)
    turn_to(target, speed)

def user_point_to_LR(pose):
    if pose is None: return 2
    if len(pose) == 0: return 2
    x0, y0 = map(int, pose[8])
    x8, y8 = map(int, pose[10])
    if x0 == x8: return 2
    k = (y0 - y8) / (x0 - x8)
    if abs(k) > 1.2: return 2
    if k > 0: return 1
    return 0

def bag_xy(dir,image):
    h, w, cc = image.shape
    result = luggage_model(image, verbose=False)[0]

    boxes = result.boxes
    
    res = zip(boxes.cls, boxes.xyxy)
    for c, xyxy in res:
        x1, y1, x2, y2 = map(int, xyxy)
        cv2.rectangle(image, (x1, y1), (x2, y2), (0, 255, 0), 2)
        b = 0
        if int((x1+x2)/2)>w//2: b = 1
        print("dir,b                           ", dir, b)
        if dir == b:
            cv2.rectangle(image, (x1, y1), (x2, y2), (255, 0, 0), 2)
            return int((x1+x2)/2),int((y1+y2)/2)
    return 0,0

def bag_any_xy(image):
    h, w, c = image.shape
    result = luggage_model(image, verbose=False)[0]

    boxes = result.boxes

    best_box = None
    best_score = -1

    for cls, xyxy in zip(boxes.cls, boxes.xyxy):
        x1, y1, x2, y2 = map(int, xyxy)

        cx = (x1 + x2) // 2
        cy = (y1 + y2) // 2

        box_area = (x2 - x1) * (y2 - y1)
        #underground
        if cy > h * 0.9:
            continue

        #score = area + if in the middle
        center_bias = abs(cx - w//2)
        score = box_area - center_bias * 200

        if score > best_score:
            best_score = score
            best_box = (x1, y1, x2, y2)

    if best_box is not None:
        x1, y1, x2, y2 = best_box
        cv2.rectangle(image, (x1, y1), (x2, y2), (255, 0, 0), 2)
        cx = (x1 + x2) // 2
        cy = (y1 + y2) // 2
        return cx, cy

    return 0, 0

def is_bag_handover_ready(image, depth):
    h, w, c = image.shape

    bagx, bagy = bag_any_xy(image)

    if bagx == 0 and bagy == 0:
        return False, 0, 0, 0

    if bagx < 0 or bagx >= w or bagy < 0 or bagy >= h:
        return False, 0, 0, 0

    bag_depth = depth[bagy][bagx]
    if bag_depth == 0:
        return False, bagx, bagy, 0
    if bag_depth > 1200:
        return False, bagx, bagy, bag_depth
    if bag_depth < 250:
        return False, bagx, bagy, bag_depth
    if bagy > h * 0.85:
        return False, bagx, bagy, bag_depth
    return True, bagx, bagy, bag_depth
def get_person_feature(image, user, api_key=OPENROUTER_API_KEY):
    img_path = save_user_crop(image, user)
    if img_path is None:
        print("save_user_crop failed")
        return None

    prompt = """
You are helping a RoboCup@Home robot identify a seated person.
Look only at the main person in the image crop.
Return JSON only, with exactly these fields:

{
  "hat": false,
  "glasses": false,
  "upper_color": "blue",
  "lower_color": "black",
  "hair_length": "short",
  "gender_like": "unclear"
}

Rules:
- Output JSON only.
- Use one-word colors.
- If uncertain, use "unclear".
- Focus only on the main person.
"""

    res = ai_vision_request(
        mode=1,
        prompt=prompt,
        api_key=api_key,
        image_path=img_path
    )

    print("vision result =", res)

    if not res["success"]:
        print("vision API failed:", res["error"])
        return None

    try:
        feat = json.loads(res["answer"])
        return feat
    except Exception as e:
        print("json parse failed:", e)
        print("raw answer:", res["answer"])
        return None

def feature_score(f1, f2):
    score = 0

    if f1 is None or f2 is None:
        return -1

    if f1["hat"] == f2["hat"]:
        score += 2
    if f1["glasses"] == f2["glasses"]:
        score += 2
    if f1["upper_color"] == f2["upper_color"]:
        score += 4
    if f1["lower_color"] == f2["lower_color"]:
        score += 3
    if f1["hair_length"] == f2["hair_length"]:
        score += 1
    if f1["gender_like"] == f2["gender_like"]:
        score += 1

    return score

def detect_person_in_view(image, depth):
    user = find_user(image, depth)
    if user is not None and len(user) > 0:
        return user
    return None

def find_user_seated(image, depth, conf_thres=0.25, min_box_area=4000):
    """
    return format:
        None 或 [best_person]

    best_person = {
        "box": (x1, y1, x2, y2),
        "center": (cx, cy),
        "depth": d,
        "conf": conf
    }
    """
    try:
        result = model(image)[0]
    except Exception as e:
        print("[find_user_seated] model error:", e)
        return None

    if result.boxes is None or len(result.boxes) == 0:
        return None

    print("find user seated:        found person")

    h, w = image.shape[:2]
    img_center_x = w / 2

    best_person = None
    best_score = -1e9

    for box in result.boxes:
        try:
            cls_id = int(box.cls[0]) if len(box.cls.shape) > 0 else int(box.cls)
            conf = float(box.conf[0]) if len(box.conf.shape) > 0 else float(box.conf)

            if cls_id != 0:
                continue
            if conf < conf_thres:
                continue

            x1, y1, x2, y2 = map(int, box.xyxy[0])
            area = (x2 - x1) * (y2 - y1)
            if area < min_box_area:
                continue

            cx = (x1 + x2) // 2
            cy = (y1 + y2) // 2

            print("depth:", depth[cy][cx])
            if depth[cy][cx] > 1500:
                continue

            x_min = max(0, cx - 10)
            x_max = min(w, cx + 10)
            y_min = max(0, cy - 10)
            y_max = min(h, cy + 10)

            patch = depth[y_min:y_max, x_min:x_max]
            vals = patch[patch > 0]
            d = float(vals.mean()) if len(vals) > 0 else -1

            depth_score = 0 if d < 0 else (-d * 0.01)
            center_score = -abs(cx - img_center_x) * 0.5
            area_score = area * 0.0005
            conf_score = conf * 100

            total_score = depth_score + center_score + area_score + conf_score

            if total_score > best_score:
                best_score = total_score
                best_person = {
                    "box": (x1, y1, x2, y2),
                    "center": (cx, cy),
                    "depth": d,
                    "conf": conf
                }

        except Exception as e:
            print("[find_user_seated] skip error:", e)
            continue

    if best_person is None:
        return None

    print("[find_user_seated] result =", best_person)
    return [best_person]

def get_person_feature_seated(image, user):
    if user is None or len(user) == 0:
        return None

    x1, y1, x2, y2 = user[0]["box"]

    crop = image[y1:y2, x1:x2]

    if crop is None or crop.size == 0:
        print("invalid crop")
        return None

    img_path = "/tmp/person.jpg"
    cv2.imwrite(img_path, crop)

    prompt = """
Describe the person.

Return JSON only:
{
  "hat": false,
  "glasses": false,
  "upper_color": "blue",
  "lower_color": "black",
  "hair_length": "short",
  "gender_like": "male"
}
"""

    result = ai_vision_request(1, prompt, API_KEY, img_path)
    print("vision result =", result)

    if not result or not result.get("success", False):
        print("vision API failed:", result.get("error", "unknown"))
        return None

    try:
        content = result["answer"]
        if content is None:
            return None

        content = content.strip()
        content = content.replace("```json", "").replace("```", "").strip()

        return json.loads(content)

    except Exception as e:
        print("feature parse error:", e)
        print("raw answer =", result["answer"])
        return None
    
def scan_people_roles(image, depth, scan_step, guest1_feature, host_feature):
    roles = []   # [(seat, role)]

    # -------- LEFT --------
    turn(+scan_step)
    rospy.sleep(1.5)

    image = _image.copy()
    depth = _depth.copy()

    user = find_user_seated(image, depth)
    if user is not None and len(user) > 0:
        role, _ = classify_person(user, image, guest1_feature, host_feature)
        roles.append(("left", role))
    else:
        roles.append(("left", "empty"))

    # -------- MIDDLE --------
    turn(-scan_step)
    rospy.sleep(1.5)

    image = _image.copy()
    depth = _depth.copy()

    user = find_user_seated(image, depth)
    if user is not None and len(user) > 0:
        role, _ = classify_person(user, image, guest1_feature, host_feature)
        roles.append(("middle", role))
    else:
        roles.append(("middle", "empty"))

    # -------- RIGHT --------
    turn(-scan_step)
    rospy.sleep(1.5)

    image = _image.copy()
    depth = _depth.copy()

    user = find_user_seated(image, depth)
    if user is not None and len(user) > 0:
        role, _ = classify_person(user, image, guest1_feature, host_feature)
        roles.append(("right", role))
    else:
        roles.append(("right", "empty"))

    # -------- 回中間 --------
    turn(+scan_step)
    rospy.sleep(1)


def classify_person(user, image, guest1_feature, host_feature, threshold=5):

    feat = get_person_feature_seated(image, user)

    print("detected feature =", feat)

    if feat is None:
        return "unknown", None

    g1_score = feature_score(feat, guest1_feature) if guest1_feature else -1
    h_score  = feature_score(feat, host_feature) if host_feature else -1

    print("g1_score =", g1_score, "host_score =", h_score)

    if g1_score >= threshold and g1_score >= h_score:
        return "guest1", feat

    if h_score >= threshold and h_score > g1_score:
        return "host", feat

    return "unknown", feat

def scan_people_roles(scan_step, guest1_feature, host_feature):
    roles = []

    global _image, _depth, _image_seq

    # -------- LEFT --------
    old_seq = _image_seq
    turn(+scan_step)
    if not wait_new_frame(old_seq, timeout=3.0):
        print("warning: no new image for left")

    image = _image.copy()
    depth = _depth.copy()

    user = find_user_seated(image, depth)
    if user is not None and len(user) > 0:
        role, feat = classify_person(user, image, guest1_feature, host_feature)
        roles.append(("left", role, feat))
    else:
        roles.append(("left", "empty", None))

    # -------- MIDDLE --------
    old_seq = _image_seq
    turn(-scan_step)
    if not wait_new_frame(old_seq, timeout=3.0):
        print("warning: no new image for middle")

    image = _image.copy()
    depth = _depth.copy()

    user = find_user_seated(image, depth)
    if user is not None and len(user) > 0:
        role, feat = classify_person(user, image, guest1_feature, host_feature)
        roles.append(("middle", role, feat))
    else:
        roles.append(("middle", "empty", None))

    # -------- RIGHT --------
    old_seq = _image_seq
    turn(-scan_step)
    if not wait_new_frame(old_seq, timeout=3.0):
        print("warning: no new image for right")

    image = _image.copy()
    depth = _depth.copy()

    user = find_user_seated(image, depth)
    if user is not None and len(user) > 0:
        role, feat = classify_person(user, image, guest1_feature, host_feature)
        roles.append(("right", role, feat))
    else:
        roles.append(("right", "empty", None))

    # -------- 回中間 --------
    old_seq = _image_seq
    turn(+scan_step)
    wait_new_frame(old_seq, timeout=2.0)

    return roles

def find_all_users(image, depth):
    global model

    h, w = depth.shape
    keypoints = model(image)[0].keypoints
    users = []

    for pose in keypoints.xy:
        mid_d = -1
        valid_points = 0

        for p in pose:
            x, y = map(int, p)
            if x >= w or y >= h or x < 0 or y < 0:
                continue
            if depth[y][x] > 0:
                valid_points += 1
                if mid_d == -1 or depth[y][x] < mid_d:
                    mid_d = depth[y][x]

        if valid_points > 0 and mid_d != -1:
            users.append((pose, mid_d))

    return users

def crop_middle_seat(image, depth):
    h, w = image.shape[:2]

    x1 = int(w * 0.30)
    x2 = int(w * 0.70)
    y1 = int(h * 0.25)
    y2 = int(h * 0.95)

    crop_img = image[y1:y2, x1:x2]
    crop_depth = depth[y1:y2, x1:x2]

    return crop_img, crop_depth

if __name__ == "__main__":
    rospy.init_node("mission1")
    rospy.loginfo("mission1 start")

    chassis = RobotChassis()
    rate = rospy.Rate(20)

    _image = None
    _depth = None
    _voice = None
    _odom = None
    _Imu = None

    _sub_cam_image, _sub_cam_depth = None, None
    _current_cam = 1

    rospy.Subscriber("/voice/text", Voice, callback_voice)
    rospy.Subscriber("/odom", Odometry, callback_odom)
    rospy.Subscriber("/respeaker/audio_path", String, callback_path)

    pub_speaker = rospy.Publisher("/speaker/say", String, queue_size=10)
    pub_cmd = rospy.Publisher("/cmd_vel", Twist, queue_size=10)

    change_to_cam(2)

    model = YOLO("yolov8n-pose.pt")
    luggage_model = YOLO("bag_model_2026.pt")

    guest1_name = ""
    guest1_drink = ""
    guest2_name = ""
    guest2_drink = ""

    host_seat = None

    empty_seat = None
    seat_names = ["left", "middle", "right"]
    point_dir = 0
    user = None
    lsp = 0
    front_angle = None
    seat_index = 0
    seat_angles = []

    guest1_feature = None
    guest2_feature = None
    host_feature = None

    guest1_now_seat = None
    host_now_seat = None

    handover_ready_count = 0
    guest2_seat = None
    guest1_seat = None
    target_bagx = 0
    target_bagy = 0
    target_bag_depth = 0
    best_guest1_score = -1
    best_host_score = -1
    host_user = None
    host_x = 0
    host_y = 0
    host_depth = -1
    host_search_count = 0
    recording_done = False
    state = 999999
    scan_step = np.pi/4
    far_right_step = 3 * np.pi / 4
    say("I am ready")

    while not rospy.is_shutdown():
        rate.sleep()

        if _image is None:
            print("image none")
            continue
        if _depth is None:
            print("depth none")
            continue
        if _odom is None:
            print("odom none")
            continue

        if len(_image) == 0 or len(_image[0]) == 0:
            continue
        if len(_depth) == 0 or len(_depth[0]) == 0:
            continue

        image = _image.copy()
        depth = _depth.copy()
        h, w, c = image.shape

        #state check
        if state == 999999:
            print("state 0: ready")
            state = 0
            
        #state0 : hear the bell ring
        if state == 0:
            print("state 0: wait for bell")

            if _audio_path is not None and os.path.exists(_audio_path):
                if detect_bell_from_file(_audio_path):
                    say("I heard the doorbell")
                    state = 1
                else:
                    print("not bell")

        #state1 : find user
        elif state == 1:
            print("state 1: find guest 1")
            user = find_user(image, depth)
            if user is not None and len(user) > 0:
                say("hello")
                feat = get_person_feature(image, user)
                print("guest1_feature =", feat)
                if feat is not None:
                    guest1_feature = feat
                    state = 2
                else:
                    print("feature failed, retry")

        #state2 : move to guest1
        elif state == 2:
            print("[STATE 2] MOVE_TO_GUEST1")
            say("detected people, ready to move")
            chassis.move_to(-0.43, -0.177, 0)
            if chassis.status_code == 3:
                print("reach guest1 interaction point")
                state = 25
            print(chassis.status_code)
        
        elif state == 25:
            turn_back()
            state = 3

        #state3 : ask guest1(name)
        elif state == 3:
            print("state 3: ask name")
            # say("what is your name")
            # rospy.sleep(1)
            guest1_name = None
            for i in range(3):
                print(f"[Name] trial {i+1}/3")
                _new_audio_flag = False
                say("what is your name")
                wav_path = wait_for_new_audio(timeout=5)
                if wav_path is None:
                    say("please repeat your name")
                    continue
                res = audio_extract_name(wav_path, OPENROUTER_API_KEY)
                print("AI name result =", res)
                if res["success"] and res["name"] is not None:
                    name = res["name"].strip()
                    if name in NAME_LIST:
                        guest1_name = name
                        break
                    else:
                        print("invalid name:", name)
                say("please repeat your name")
            # fallback
            if guest1_name is None:
                guest1_name = "friend"
            print("guest1_name =", guest1_name)
            say(f"hello {guest1_name}")
            state = 4

        #state4 : ask guest1(drink)
        elif state == 4:
            print("state 4: ask drink")
            # say("what is your favorite drink")
            # rospy.sleep(1)
            guest1_drink = None
            for i in range(3):
                print(f"[Drink] trial {i+1}/3")
                _new_audio_flag = False
                say("what is your favorite drink")
                wav_path = wait_for_new_audio(timeout=5)
                if wav_path is None:
                    say("please repeat your drink")
                    continue
                res = audio_extract_drink(wav_path, OPENROUTER_API_KEY)
                print("AI drink result =", res)
                if res["success"] and res["drink"] is not None:
                    drink = res["drink"].strip().lower()
                    valid_drinks = [d.lower() for d in DRINK_LIST]
                    if drink in valid_drinks:
                        guest1_drink = DRINK_LIST[valid_drinks.index(drink)]
                        break
                    else:
                        print("invalid drink:", drink)
                say("please repeat your drink")
            # fallback
            if guest1_drink is None:
                guest1_drink = "water"  
            print("guest1_drink =", guest1_drink)
            say(f"{guest1_name} likes {guest1_drink}")
            state = 5

        #state5 : seat guest 
        elif state == 5:
            print("state 5: seat guest 1")
            say("please follow me")
            state = 6

        #state 6 : turn back
        elif state == 6:
            turn_back()      
            print("success turning 180")
            state = 7

        #state 7 : move to point 1
        elif state == 7:
            print("starting to move to point 1")
            chassis.move_to(1.01, -0.305, 0)
            if chassis.status_code == 3:
                print("reach point 1")
                state = 8
        
        #state 8 : move to main point
        elif state == 8:
            print("state 8: move to main point")
            chassis.move_to(1.14, 2.2, 0)
            if chassis.status_code == 3:
                say("reach main point")
                front_angle = get_yaw_from_odom()
                print("front_angle =", front_angle)
                seat_names = ["left_chair", "sofa_left", "sofa_right", "right_chair"]
                seat_angles = [
                    normalize_angle(front_angle + np.pi/4),    # left chair
                    normalize_angle(front_angle + np.pi/10),   # sofa left
                    normalize_angle(front_angle - np.pi/10),   # sofa right
                    normalize_angle(front_angle - np.pi/3)     # right chair / host side
                ]
                seat_index = 0
                empty_seat = None
                guest1_seat = None
                host_seat = None
                host_feature = None
                state = 9

        #elif state == 85:
            #turn_back()
            #turn(10/180*np.pi)
            #print("state85: turn back and turn 10")
            #state=9
        
        #state 9 : find empty seat first, then find host
        elif state == 9:
            print("state 9: find empty chair for guest1")

            empty_seat = None
            guest1_seat = None

            # -------- LEFT --------
            old_seq = _image_seq
            turn(+scan_step)
            wait_new_frame(old_seq)

            image = _image.copy()
            depth = _depth.copy()
            user = find_user_seated(image, depth)

            if user is None or len(user) == 0:
                print("left empty")
                empty_seat = "left"
                guest1_seat = "left"

                say("this is your seat")
                rospy.sleep(2)

            else:
                print("left occupied")

                # -------- MIDDLE --------
                old_seq = _image_seq
                turn(-scan_step)
                wait_new_frame(old_seq)

                image = _image.copy()
                depth = _depth.copy()

                middle_img, middle_depth = crop_middle_seat(image, depth)
                user = find_user_seated(middle_img, middle_depth)

                if user is None or len(user) == 0:
                    print("middle empty")
                    empty_seat = "middle"
                    guest1_seat = "middle"

                    say("this is your seat")
                    rospy.sleep(2)

                else:
                    print("middle occupied")

                    # -------- RIGHT --------
                    old_seq = _image_seq
                    turn(-scan_step)
                    wait_new_frame(old_seq)

                    image = _image.copy()
                    depth = _depth.copy()
                    user = find_user_seated(image, depth)

                    if user is None or len(user) == 0:
                        print("right empty")
                        empty_seat = "right"
                        guest1_seat = "right"

                        say("this is your seat")
                        rospy.sleep(2)

                    else:
                        print("right occupied")

                        # -------- FAR RIGHT --------
                        old_seq = _image_seq
                        turn(-far_right_step)
                        wait_new_frame(old_seq)

                        image = _image.copy()
                        depth = _depth.copy()
                        user = find_user_seated(image, depth)
                        print("checking far right seat")
                        if user is None or len(user) == 0:
                            print("far_right empty")
                            empty_seat = "far_right"
                            guest1_seat = "far_right"

                            say("this is your seat")
                            rospy.sleep(2)

                        else:
                            print("far_right occupied")

            # -------- 回 MIDDLE --------
            old_seq = _image_seq
            turn(+far_right_step + scan_step)
            wait_new_frame(old_seq)

            print("guest1_seat =", guest1_seat)

            if guest1_seat is None:
                say("sorry, no empty chair found")

            state = 10

        # state10 : turn back to front
        elif state == 10:
            print("state 10: turn back to front")
            turn_back()
            print("guest1_name =", guest1_name)
            print("guest1_drink =", guest1_drink)
            print("guest1_feature =", guest1_feature)
            print("guest1_seat =", guest1_seat)
            print("host_seat =", host_seat)
            print("host_feature =", host_feature)
            state = 105
        
        elif state == 105:
            print("[STATE 10.5] MOVE_TO_GUEST1")
            say("detected people, ready to move")
            chassis.move_to(1.01, -0.305, 0)
            if chassis.status_code == 3:
                print("reach guest1 interaction point")
                state = 11
            print(chassis.status_code)
        
#state two : find guest 2   
        #state 11 : find guest 2
        elif state == 11:
            print("state 11: find guest 2")
            user = find_user(image, depth)
            if user is not None and len(user) > 0:
                say("hello")
                feat = get_person_feature(image, user)
                print("guest2_feature =", feat)
                if feat is not None:
                    guest2_feature = feat
                    state = 12
                else:
                    print("feature failed, retry")

        #state = 12: navigate to point
        elif state == 12:
            print("[STATE 12] MOVE_TO_GUEST1")
            say("detected people, ready to move")
            chassis.move_to(-0.43, -0.177, 0)
            if chassis.status_code == 3:
                print("reach guest1 interaction point")
                state = 125
            print(chassis.status_code)
        
        elif state == 125:
            turn_back()
            state = 13

        #state 13 : ask guest 2 name
        elif state == 13:
            print("state 13: ask name")
            # say("what is your name")
            # rospy.sleep(1)
            guest1_name = None
            for i in range(3):
                print(f"[Name] trial {i+1}/3")
                _new_audio_flag = False
                say("what is your name")
                wav_path = wait_for_new_audio(timeout=5)
                if wav_path is None:
                    say("please repeat your name")
                    continue
                res = audio_extract_name(wav_path, OPENROUTER_API_KEY)
                print("AI name result =", res)
                if res["success"] and res["name"] is not None:
                    name = res["name"].strip()
                    if name in NAME_LIST:
                        guest2_name = name
                        break
                    else:
                        print("invalid name:", name)
                say("please repeat your name")
            # fallback
            if guest2_name is None:
                guest2_name = "friend"
            print("guest2_name =", guest2_name)
            say(f"hello {guest2_name}")
            state = 14

        #state14 : ask guest2(drink)
        elif state == 14:
            print("state 14: ask drink")
            # say("what is your favorite drink")
            # rospy.sleep(1)
            guest2_drink = None
            for i in range(3):
                print(f"[Drink] trial {i+1}/3")
                _new_audio_flag = False
                say("what is your favorite drink")
                wav_path = wait_for_new_audio(timeout=5)
                if wav_path is None:
                    say("please repeat your drink")
                    continue
                res = audio_extract_drink(wav_path, OPENROUTER_API_KEY)
                print("AI drink result =", res)
                if res["success"] and res["drink"] is not None:
                    drink = res["drink"].strip().lower()
                    valid_drinks = [d.lower() for d in DRINK_LIST]
                    if drink in valid_drinks:
                        guest2_drink = DRINK_LIST[valid_drinks.index(drink)]
                        break
                    else:
                        print("invalid drink:", drink)
                say("please repeat your drink")
            # fallback
            if guest2_drink is None:
                guest2_drink = "water"  
            print("guest2_drink =", guest2_drink)
            say(f"{guest2_name} likes {guest2_drink}")
            state = 155

        elif state == 155:
            say("please put your bag on my robot arm in five seconds thank you")
            state = 15

        #state 15: seat guest 2
        elif state == 15:
            print("state 15: seat guest 2")
            say("please follow me")
            state = 16

        #state 16: turn 180
        elif state == 16:
            turn_back()
            print("success turning 180")
            state = 17
       
        #state 17: move to point 1
        elif state == 17:
            print("starting to move to point 1")
            chassis.move_to(1.7, -0.118, 0.00156)
            if chassis.status_code == 3:
                print("reach point 1")
                state = 18

        #state 18: move to main point
        elif state == 18:
            print("state 18: move to main point")
            chassis.move_to(1.14, 2.2, 0)

            if chassis.status_code == 3:
                say("reach main point")

                empty_seat = None
                guest2_seat = None
                guest1_now_seat = None

                state = 19

        # scan seats (loop state)
        # state 19 : scan chairs, and check where guest1 is every turn
        elif state == 19:
            print("state 19: find guest1 and empty seat for guest2")

            empty_seat = None
            guest1_now_seat = None

            # -------- LEFT --------
            old_seq = _image_seq
            turn(+scan_step)
            wait_new_frame(old_seq)

            image = _image.copy()
            depth = _depth.copy()

            middle_img, middle_depth = crop_middle_seat(image, depth)
            user = find_user_seated(middle_img, middle_depth)

            if user is not None and len(user) > 0:
                print("left occupied, this is guest1")
                guest1_now_seat = "left"
            else:
                print("left empty")
                if empty_seat is None:
                    empty_seat = "left"

            # -------- MIDDLE --------
            old_seq = _image_seq
            turn(-scan_step)
            wait_new_frame(old_seq)

            image = _image.copy()
            depth = _depth.copy()
            user = find_user_seated(image, depth)

            if user is not None and len(user) > 0:
                print("middle occupied, this is guest1")
                guest1_now_seat = "middle"
            else:
                print("middle empty")
                if empty_seat is None:
                    empty_seat = "middle"

            # -------- RIGHT --------
            old_seq = _image_seq
            turn(-scan_step + 10/180*np.pi)
            wait_new_frame(old_seq)

            image = _image.copy()
            depth = _depth.copy()
            user = find_user_seated(image, depth)

            if user is not None and len(user) > 0:
                print("right occupied, this is guest1")
                guest1_now_seat = "right"
            else:
                print("right empty")
                if empty_seat is None:
                    empty_seat = "right"

            # -------- FAR RIGHT --------
            old_seq = _image_seq
            turn(-far_right_step)
            wait_new_frame(old_seq)

            image = _image.copy()
            depth = _depth.copy()
            user = find_user_seated(image, depth)

            if user is not None and len(user) > 0:
                print("far_right occupied, this is guest1")
                guest1_now_seat = "far_right"
            else:
                print("far_right empty")
                if empty_seat is None:
                    empty_seat = "far_right"

            # -------- 回 MIDDLE --------
            old_seq = _image_seq
            turn(+far_right_step + scan_step)
            wait_new_frame(old_seq)

            print("guest1_now_seat =", guest1_now_seat)
            print("empty_seat =", empty_seat)

            state = 20

        elif state == 20:
            print("state 20: seat guest2")

            if empty_seat is None:
                say("sorry, no empty chair found")
                guest2_seat = None
                state = 21

            else:
                guest2_seat = empty_seat
                print("guest2_seat =", guest2_seat)

                if guest2_seat == "left":
                    turn(+scan_step)

                elif guest2_seat == "middle":
                    pass

                elif guest2_seat == "right":
                    turn(-scan_step + 10/180*np.pi)

                elif guest2_seat == "far_right":
                    turn(-scan_step + 10/180*np.pi)
                    rospy.sleep(0.5)
                    turn(-far_right_step)

                rospy.sleep(1)
                say("this is your seat")
                rospy.sleep(2)

                state = 21

        elif state == 21:
            print("state 21: turn to host and start follow me")

            # 先回 middle，因為 state 20 可能面向 guest2 seat
            if guest2_seat == "left":
                turn(-scan_step)

            elif guest2_seat == "middle":
                pass

            elif guest2_seat == "right":
                turn(+scan_step - 10/180*np.pi)

            elif guest2_seat == "far_right":
                turn(+far_right_step)
                rospy.sleep(0.5)
                turn(+scan_step - 10/180*np.pi)

            rospy.sleep(1)

            # middle -> right -> far_right(host)
            turn(-scan_step + 10/180*np.pi)
            rospy.sleep(0.5)
            turn(-far_right_step)
            rospy.sleep(1)

            say("host, please guide me")
            rospy.sleep(1)

            follow()

            state = 22




#changed in 28/5/2026
