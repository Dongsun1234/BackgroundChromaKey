import cv2 as cv
import numpy as np
import random
import spidev
import time
import datetime

cap = cv.VideoCapture(0)
cap.set(cv.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv.CAP_PROP_FRAME_HEIGHT, 480)
# retval, frame = cap.read()

spi = spidev.SpiDev()
spi.open(0, 0)
spi.max_speed_hz = 100000

delay = 0.3

sw_channel = 0
vrx_channel = 1
vry_channel = 2


def imagechange(vrx_pos, vry_pos, sw_val):
    if sw_val == 0:
        img = np.full(
            (640, 480, 3),
            (
                random.randrange(0, 256),
                random.randrange(0, 256),
                random.randrange(0, 256),
            ),
            dtype=np.uint8,
        )  # 사이즈 크기, 색상BGR 랜덤, 0~255까지 정수 자료형 타입
    elif (vrx_pos > 530 and vrx_pos <= 1023) and (
        vry_pos > 530 and vry_pos <= 1023
    ):  # 조이스틱 좌표를 통한 배경 이미지 설정
        img = cv.imread("./background.png")
    elif (vrx_pos < 490 and vrx_pos >= 0) and (vry_pos > 530 and vry_pos <= 1023):
        img = cv.imread("./background1.png")
    elif (vrx_pos < 490 and vrx_pos >= 0) and (vry_pos >= 0 and vry_pos < 490):
        img = cv.imread("./background2.png")
    elif (vrx_pos > 530 and vrx_pos <= 1023) and (vry_pos >= 0 and vry_pos < 490):
        img = cv.imread("./background3.png")
    else:
        img = cv.imread("./background4.png")
    img = cv.resize(img, (640, 480))
    return img


def readadc(adcnum):
    if adcnum > 7 or adcnum < 0:
        return -1
    r = spi.xfer2([1, (8 + adcnum) << 4, 0])
    data = ((r[1] & 3) << 8) + r[2]
    return data


def imagefilter():
    while True:
        retval, frame = cap.read()

        gray_frame = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
        ret, frame_threshold = cv.threshold(gray_frame, 70, 255, cv.THRESH_BINARY)
        # 머리카락 검정 배경 흰색
        if not retval and not ret:
            break
        sw_val = readadc(sw_channel)
        vrx_pos = readadc(vrx_channel)
        vry_pos = readadc(vry_channel)
        img = imagechange(vrx_pos, vry_pos, sw_val)

        hsv = cv.cvtColor(frame, cv.COLOR_BGR2HSV)

        brown_lo = np.array([39, 0, 50])  
        brown_hi = np.array([255, 255, 255]) 

        mask_brown = cv.inRange(hsv, brown_lo, brown_hi)  # 얼굴 검 배경 흰
        mask_brown_inv = cv.bitwise_not(mask_brown)  # 얼굴 흰 배경 검정

        k = cv.getStructuringElement(cv.MORPH_RECT, (3, 3))

        # 모폴로지 1,1 커널 크기 지정 (배경에 노이즈를 없애기 위해)
        output_mor = cv.erode(mask_brown_inv, k)  # 커널 안에 전부 1이 있지 않으면 0으로 처리
        output_mor_inv = cv.bitwise_not(output_mor)  # 얼굴 검정, 배경 흰색으로 다시 변환

        blackface = cv.bitwise_and(frame_threshold, output_mor_inv)  # 얼굴 검 배경 흰
        blackface_inv = cv.bitwise_not(blackface)  # 얼굴이 흰색 배경 검정

        masked_frame = cv.bitwise_and(frame, frame, mask=blackface_inv)  # 얼굴 원본 배경 검정
        masked_img = cv.bitwise_and(img, img, mask=blackface)
        # 원본 이미지를 and 연산하여 얼굴이 검정과 배경이 이미지로 연산
        result = cv.add(masked_frame, masked_img)  # 이미지 더하기(얼굴 촬영+배경 지정)
        draw = result.copy()  # 결과 복사

        print("-------------")
        print(
            "X : %d  Y: %d SW: %d" % (vrx_pos, vry_pos, sw_val)
        )  # 조이스틱의 X축, Y축, SW 값 출력
        time.sleep(delay)

        cv.imshow("result", result)
        key = cv.waitKey(25)
        if key == 27:  # esc 누르면 종료
            break
        if key == 113:  # q 누르면 저장
            suffix = datetime.datetime.now().strftime("%y%m%d_%H%M%S")  # 현재 시간으로 저장
            cv.imwrite("/home/pi/Pictures/" + suffix + ".jpg", draw)  # 저장 경로 및 복사 이미지

    if cap.isOpened():
        cap.release()

    cv.destroyAllWindows()


if __name__ == "__main__":
    imagefilter()
