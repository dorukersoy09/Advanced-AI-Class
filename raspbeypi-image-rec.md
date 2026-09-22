# Image Recognission Project

## 1. Raspberry Pi OS Installation

We first downloaded **Raspberry Pi Imager** from the official Raspberry Pi website:

[Raspberry Pi Imager](https://www.raspberrypi.com/software/?utm_source=chatgpt.com)

Raspberry Pi Imager is used to install Raspberry Pi OS and other operating systems onto a microSD card.

### Setting up the SD card

1. We opened **Raspberry Pi Imager**.
2. We selected our Raspberry Pi model:

   * **Raspberry Pi 4**
3. We selected:

   * **Raspberry Pi OS 64-bit**
4. We selected the **SD card** that would be inserted into the Raspberry Pi.
5. On the personalization page, we **skipped the personalization settings**.
6. We pressed **Write** to flash the operating system onto the SD card.

### First Raspberry Pi startup

After the SD card was flashed:

1. We inserted the SD card into the Raspberry Pi.
2. We connected the Raspberry Pi to a monitor.
3. We started the Raspberry Pi and used the graphical interface.
4. We selected and adjusted the appropriate **country and region settings**.
5. We used the default login credentials:

   * Username: `pi`
   * Password: `raspberry`
6. We waited for Raspberry Pi OS to finish installing the latest updates.

---

# 2. Connecting to the Raspberry Pi Through SSH

After setting up the Raspberry Pi using the monitor, we wanted to control it directly from the Mac terminal.

The Raspberry Pi's local IP address was:

```text
192.168.68.106
```

From the Mac terminal, we connected using:

```bash
ssh pi@192.168.68.106
```

The Raspberry Pi terminal then showed:

```text
pi@raspberrypi:~ $
```

This allowed us to perform the rest of the project without needing the monitor.

---

# 3. Checking the Raspberry Pi

We checked the Raspberry Pi's operating system and hardware architecture.

The system reported:

```text
Linux raspberrypi 6.18.50+rpt-rpi-v8
```

The operating system was:

```text
Debian GNU/Linux 13 (trixie)
```

The architecture was:

```text
aarch64
```

This confirmed that we were running a **64-bit ARM system**.

We also checked the available storage:

```bash
df -h /
```

The Raspberry Pi had approximately:

```text
28 GB total
6.6 GB used
21 GB available
```

---

# 4. Creating the Project

We created a dedicated directory for the image recognition project:

```bash
mkdir -p ~/projects/raspberry-object-detection
```

Inside the project we created:

```text
raspberry-object-detection/
├── images/
├── models/
├── results/
└── src/
```

The purpose of each directory is:

| Directory  | Purpose                             |
| ---------- | ----------------------------------- |
| `images/`  | Captured camera images              |
| `models/`  | Object detection models and labels  |
| `results/` | Images containing detection results |
| `src/`     | Python source code                  |

---

# 5. Python Virtual Environment

We checked the Python version:

```bash
python3 --version
```

The Raspberry Pi was running:

```text
Python 3.13.5
```

We created a Python virtual environment:

```bash
python3 -m venv --system-site-packages .venv
```

We activated it using:

```bash
source .venv/bin/activate
```

When activated, the terminal showed:

```text
(.venv) pi@raspberrypi:~/projects/raspberry-object-detection $
```

The `--system-site-packages` option was important because we wanted the virtual environment to be able to access Raspberry Pi system packages such as Picamera2.

---

# 6. Installing the Raspberry Pi Camera Software

We installed Picamera2:

```bash
sudo apt install -y python3-picamera2
```

We then recreated the virtual environment with access to the system packages:

```bash
rm -rf .venv
python3 -m venv --system-site-packages .venv
source .venv/bin/activate
```

---

# 7. Raspberry Pi Camera Module 3

The project uses a **Raspberry Pi Camera Module 3**.

We checked whether Raspberry Pi OS could detect the camera:

```bash
rpicam-hello --list-cameras
```

The camera was detected as:

```text
imx708
```

The available camera modes included:

```text
1536x864
2304x1296
4608x2592
```

This confirmed that the Camera Module 3 was correctly connected and recognized.

---

# 8. Testing the Camera

We first tested the camera directly:

```bash
rpicam-still -o test.jpg
```

The camera successfully captured an image.

We then created a Python camera test:

```bash
nano src/camera.py
```

The code was:

```python
from picamera2 import Picamera2
import time

camera = Picamera2()

config = camera.create_still_configuration(
    main={"size": (2304, 1296)}
)

camera.configure(config)

camera.start()

time.sleep(2)

camera.capture_file("images/test_python.jpg")

camera.stop()

print("Photo captured successfully!")
```

We ran it using:

```bash
python3 src/camera.py
```

The resulting image was saved to:

```text
images/test_python.jpg
```

We could copy it from the Raspberry Pi to the Mac using:

```bash
scp pi@192.168.68.106:~/projects/raspberry-object-detection/images/test_python.jpg ~/Desktop/
```

---

# 9. Choosing the Object Detection Framework

We initially attempted to install Ultralytics:

```bash
pip install ultralytics
```

However, this attempted to install a large PyTorch and NVIDIA CUDA dependency stack.

The installation eventually failed with:

```text
OSError: [Errno 28] No space left on device
```

We therefore decided not to use Ultralytics on this Raspberry Pi setup.

We also tested ONNX Runtime:

```bash
python3 -c "import onnxruntime; print(onnxruntime.__version__)"
```

However, importing the installed ONNX Runtime resulted in:

```text
Illegal instruction
```

Therefore, we did not use ONNX Runtime either.

---

# 10. Using LiteRT

We installed Google's LiteRT package:

```bash
pip install ai-edge-litert
```

We tested the installation:

```bash
python3 -c "import ai_edge_litert; print('LiteRT OK')"
```

The result was:

```text
LiteRT OK
```

Therefore, LiteRT became the inference engine for our object detection system.

---

# 11. Downloading the Object Detection Model

We downloaded the **COCO SSD MobileNet v1** model.

The model was:

```text
coco_ssd_mobilenet_v1_1.0_quant_2018_06_29.zip
```

It was downloaded from:

```text
https://storage.googleapis.com/download.tensorflow.org/models/tflite/coco_ssd_mobilenet_v1_1.0_quant_2018_06_29.zip
```

The model was extracted into:

```text
models/
```

The important model file is:

```text
models/detect.tflite
```

---

# 12. Object Detection Labels

The model uses a COCO label map.

The label file is:

```text
models/labelmap.txt
```

Some of the objects the model can recognize include:

```text
person
bicycle
car
motorcycle
airplane
bus
train
truck
boat
traffic light
stop sign
bench
bird
cat
dog
horse
sheep
cow
elephant
bear
zebra
giraffe
backpack
umbrella
handbag
tie
suitcase
frisbee
skis
snowboard
sports ball
kite
baseball bat
baseball glove
skateboard
surfboard
tennis racket
bottle
wine glass
cup
fork
knife
spoon
bowl
banana
apple
sandwich
orange
broccoli
carrot
hot dog
pizza
donut
cake
chair
couch
potted plant
bed
dining table
toilet
tv
laptop
mouse
remote
keyboard
cell phone
microwave
oven
toaster
sink
refrigerator
book
clock
vase
scissors
teddy bear
hair drier
toothbrush
```

---

# 13. First Object Detection Program
<img width="640" height="480" alt="detection" src="https://github.com/user-attachments/assets/3f9bc335-2f8f-4c30-9101-ec24b04ef608" />

We created:

```text
src/detect.py
```

The first version of the detector:

1. Loaded the LiteRT model.
2. Started the Camera Module 3.
3. Captured one image.
4. Resized it to the model's input size.
5. Ran object detection.
6. Drew bounding boxes around detected objects.
7. Saved the result to:

```text
results/detection.jpg
```

The model reported:

```text
Model input: 300x300
```

We successfully ran the detector.

When there was nothing interesting in front of the camera, it reported:

```text
Detected 0 object(s)
```

We then tested it with a computer in front of the camera.

The detector identified the computer screen as:

```text
tv: 66%
```

This confirmed that the **complete object detection pipeline was working**.

The classification of the computer screen as a TV is understandable because the COCO model has a `tv` class but does not have a dedicated class for every type of computer monitor.

---

# 14. Moving Toward Real-Time Detection

The original detector captured only one image.

We then changed the program so that it continuously:

```text
Capture frame
      ↓
Resize frame
      ↓
Run object detection
      ↓
Find objects
      ↓
Draw bounding boxes
      ↓
Repeat
```

The Raspberry Pi was successfully able to start the continuous detection loop.

---

# 15. Real-Time Detection Code

The current `src/detect.py` uses the following code:

```python
from picamera2 import Picamera2
from ai_edge_litert.interpreter import Interpreter

import cv2
import numpy as np
import time


MODEL = "models/detect.tflite"
LABELS = "models/labelmap.txt"

CONFIDENCE = 0.50


def load_labels(filename):
    labels = []

    with open(filename, "r") as file:
        for line in file:
            label = line.strip()

            if label:
                labels.append(label)

    return labels


print("Loading model...")

labels = load_labels(LABELS)

interpreter = Interpreter(
    model_path=MODEL,
    num_threads=4
)

interpreter.allocate_tensors()

input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()

input_height = input_details[0]["shape"][1]
input_width = input_details[0]["shape"][2]

print(f"Model input: {input_width}x{input_height}")


# Start camera
camera = Picamera2()

config = camera.create_preview_configuration(
    main={
        "size": (640, 480),
        "format": "RGB888"
    }
)

camera.configure(config)
camera.start()

time.sleep(2)

print()
print("Real-time detection started.")
print("Press Ctrl+C to quit.")
print()


try:

    while True:

        # Capture frame
        image = camera.capture_array()

        # Resize for the model
        resized = cv2.resize(
            image,
            (input_width, input_height)
        )

        # Add batch dimension
        input_data = np.expand_dims(resized, axis=0)

        # Handle model input type
        if input_details[0]["dtype"] == np.float32:
            input_data = (
                np.float32(input_data) - 127.5
            ) / 127.5

        # Run model
        interpreter.set_tensor(
            input_details[0]["index"],
            input_data
        )

        interpreter.invoke()

        # Get results
        boxes = interpreter.get_tensor(
            output_details[0]["index"]
        )

        classes = interpreter.get_tensor(
            output_details[1]["index"]
        )

        scores = interpreter.get_tensor(
            output_details[2]["index"]
        )

        num_detections = interpreter.get_tensor(
            output_details[3]["index"]
        )

        height, width = image.shape[:2]

        detections = 0

        # Process detections
        for i in range(int(num_detections[0])):

            score = float(scores[0][i])

            if score < CONFIDENCE:
                continue

            class_id = int(classes[0][i])

            if 0 <= class_id < len(labels):
                name = labels[class_id]
            else:
                name = f"class {class_id}"

            top, left, bottom, right = boxes[0][i]

            x1 = max(0, int(left * width))
            y1 = max(0, int(top * height))
            x2 = min(width, int(right * width))
            y2 = min(height, int(bottom * height))

            # Draw bounding box
            cv2.rectangle(
                image,
                (x1, y1),
                (x2, y2),
                (0, 255, 0),
                2
            )

            # Draw label
            text = f"{name}: {score:.0%}"

            cv2.putText(
                image,
                text,
                (x1, max(25, y1 - 10)),
                cv2.FONT_HERSHEY_SIMPLEX,
                0.6,
                (0, 255, 0),
                2
            )

            detections += 1

        # Save latest frame
        cv2.imwrite(
            "results/latest.jpg",
            image
        )


except KeyboardInterrupt:

    print()
    print("Stopping detection...")


finally:

    camera.stop()
    print("Camera stopped.")
```

---

# 16. Camera Format Fix

When we first changed the program to real-time detection, the camera produced a 4-channel image:

```text
640x480-XBGR8888
```

The object detection model expected a 3-channel image.

This produced:

```text
ValueError: Cannot set tensor: Dimension mismatch.
Got 4 but expected 3 for dimension 3
```

We fixed this by explicitly configuring the camera to output:

```python
"format": "RGB888"
```

The relevant section is:

```python
config = camera.create_preview_configuration(
    main={
        "size": (640, 480),
        "format": "RGB888"
    }
)
```

After this change, the camera frames could be passed to the model correctly.

---

# 17. Current Project Status

At this point we have successfully completed:

* [x] Raspberry Pi OS installation
* [x] Raspberry Pi 4 setup
* [x] 64-bit Raspberry Pi OS
* [x] Camera Module 3 setup
* [x] SSH connection from Mac
* [x] Python environment
* [x] Picamera2
* [x] Camera capture
* [x] LiteRT installation
* [x] Object detection model
* [x] COCO labels
* [x] Single-image object detection
* [x] Successful object recognition
* [x] Real-time detection loop
* [x] Camera format compatibility fix

The current system can continuously capture frames and run object detection.

---

# 18. Next Step

The next planned improvement is to create a **live camera feed accessible from the Mac**.

The intended architecture is:

```text
             Raspberry Pi
        ┌─────────────────────┐
        │  Camera Module 3    │
        │         ↓           │
        │  Camera Frame       │
        │         ↓           │
        │  LiteRT Detection   │
        │         ↓           │
        │ Bounding Boxes       │
        │         ↓           │
        │  Video Stream       │
        └──────────┬──────────┘
                   │
                Wi-Fi
                   │
                   ↓
              MacBook Pro
                   │
                Browser
                   │
                   ↓
          Live Detection Feed
```

This will allow us to see the camera feed on the Mac while the Raspberry Pi performs the object detection.

After the live feed is working, the project can progress toward:

1. Improving detection speed.
2. Saving detected images.
3. Recording detection events.
4. Downloading images from the Raspberry Pi.
5. Selecting objects that the standard COCO model cannot recognize.
6. Collecting a custom dataset.
7. Training a custom object-detection model.
8. Deploying the custom model back onto the Raspberry Pi.
9. Eventually creating a complete standalone object-recognition system.
