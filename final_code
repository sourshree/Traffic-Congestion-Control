from transformers import DetrImageProcessor, DetrForObjectDetection
import torch
from PIL import Image
import time
import cv2

processor = DetrImageProcessor.from_pretrained("facebook/detr-resnet-50", revision="no_timm")
model = DetrForObjectDetection.from_pretrained("facebook/detr-resnet-50", revision="no_timm")
lights = [30 , -30] #using + to denote red lights and - to denote green
frame=0

def process_frame(frame):
    pil_image = Image.fromarray(cv2.cvtColor(frame, cv2.COLOR_BGR2RGB))
    inputs = processor(images=pil_image, return_tensors="pt")
    outputs = model(**inputs)
    target_sizes = torch.tensor([pil_image.size[::-1]])
    results = processor.post_process_object_detection(outputs, target_sizes=target_sizes, threshold=0.9)[0]
    traffic = ['car', 'scooter', 'bus', 'motorcycle', 'truck', 'bicycle', 'van']
    return sum(1 for label in results["labels"] if model.config.id2label[label.item()] in traffic)

def traffic(path='video.mp4', frame_num=0):
    video_path = path
    vidcap = cv2.VideoCapture(video_path)
    vidcap.set(cv2.CAP_PROP_POS_FRAMES, frame_num)
    try:
        success, image = vidcap.read()
        if not success:
            return 0
        cars=process_frame(image)
        print(cars)
        return cars
    except Exception as e:
        print("An error occurred:", e)
        return 0

def lights_status(cam1='cam1.mp4', cam2='cam2.mp4',frame=frame, lights=lights):
    while True:
        count1=traffic(cam1, frame)
        count2=traffic(cam2, frame)
        if lights[0]>0 and count1-count2>15:
            lights[0] = lights[0] - lights[0] // 4
            lights[1] = -lights[0]
        elif lights[1]>0 and count2-count1>15:
            lights[1] = lights[1] - lights[1] // 4
            lights[0] = -lights[1]
        else:
            if lights[0]>0:
                lights[0],lights[1]=lights[0]-1,lights[1]+1
            elif lights[1]>0:
                lights[0],lights[1]=lights[0]+1,lights[1]-1
        if lights==[0,0]:
            if count1>count2:
                lights = [-60,+60]
            else:
                lights=[60,-60]
        print(lights)
        frame += 24
        time.sleep(0.5)
        
if __name__ == "__main__":
    lights_status()

