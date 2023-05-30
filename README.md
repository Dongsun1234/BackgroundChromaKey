```python
brown_lo = np.array([39, 0, 50])  # 39,0,55  h,s,v
brown_hi = np.array([255, 255, 255])  # 86, 255, 255

mask_brown = cv.inRange(hsv, brown_lo, brown_hi)  # 얼굴 검 배경 흰
mask_brown_inv = cv.bitwise_not(mask_brown)  # 얼굴 흰 배경 검정 

cv.imshow("result", mask_brown) #mask_brown_inv
key = cv.waitKey(25)
    if key == 27:  # esc 누르면 종료
        break

```

![mask_brown](https://github.com/Dongsun1234/BackgroundChromaKey/assets/130419965/11238559-f262-4884-8ccb-e542d7165fe7)

### 하지만 몸 영역 외의 다른 배경에서도 같이 검출이 되는 것을 볼 수 있었다. 그리고 머리카락 영역에서도 완전히 검출되지 않았다.

```python
brown_lo = np.array([39, 0, 50])  
brown_hi = np.array([255, 255, 255]) 

mask_brown = cv.inRange(hsv, brown_lo, brown_hi)  # 얼굴 검 배경 흰
mask_brown_inv = cv.bitwise_not(mask_brown)  # 얼굴 흰 배경 검정 

k = cv.getStructuringElement(cv.MORPH_RECT, (3, 3))
# 모폴로지 3,3 커널 크기 지정 (배경에 노이즈를 없애기 위해)
output_mor = cv.erode(mask_brown_inv, k)  # 커널 안에 전부 1이 있지 않으면 0으로 처리
output_mor_inv = cv.bitwise_not(output_mor)  # 얼굴 검정, 배경 흰색으로 다시 변환

cv.imshow("result", output_mor_inv) #output_mor
key = cv.waitKey(25)
if key == 27:  # esc 누르면 종료
    break
```


![mask_mor_inv](https://github.com/Dongsun1234/BackgroundChromaKey/assets/130419965/b9f11133-e2bb-46d2-95c9-b34200bc8f16)

### 모폴로지를 통해  커널 크기 만큼 1이 있지 않으면 0으로 처리하여 이미지 필터를 적용하였다.

```python
retval, frame = cap.read()

if not retval:
    break
gray_frame = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
ret, frame_threshold = cv.threshold(gray_frame, 70, 255, cv.THRESH_BINARY)
# 머리카락 검정 배경 흰색
if not ret:
    break
#머리카락 영역이 제대로 잡히지 않아 프레임을 이진화하여 해당 머리카락을 추출하게 하였다.

blackface = cv.bitwise_and(frame_threshold, output_mor_inv)  # 얼굴 검 배경 흰
blackface_inv = cv.bitwise_not(blackface)  # 얼굴이 흰색 배경 

cv.imshow("result", blackface) #blackface_inv
key = cv.waitKey(25)
if key == 27:  # esc 누르면 종료
    break
```
![blackface](https://github.com/Dongsun1234/BackgroundChromaKey/assets/130419965/e63ed18a-6b57-4cad-9bc6-0c21282f28d8)
![blackface_inv](https://github.com/Dongsun1234/BackgroundChromaKey/assets/130419965/f3d8f853-89a2-466f-af28-b072381d732e)

```python
masked_frame = cv.bitwise_and(frame, frame, mask=blackface_inv)  # 얼굴 원본 배경 검정
masked_img = cv.bitwise_and(img, img, mask=blackface)
# 원본 이미지를 and 연산하여 얼굴이 검정과 배경이 이미지로 연산
result = cv.add(masked_frame, masked_img)  # 이미지 더하기(얼굴 촬영+배경 지정)
draw = result.copy()  # 결과 복사

cv.imshow("result", mask_brown_inv)
key = cv.waitKey(25)
if key == 27:  # esc 누르면 종료
    break
# 이미지 출력
```
![masked_img](https://github.com/Dongsun1234/BackgroundChromaKey/assets/130419965/04b68214-403a-4903-9943-270b6b6b41c9)
![모자이크](https://github.com/Dongsun1234/BackgroundChromaKey/assets/130419965/9bc09ec7-af0b-4a49-b3bc-63dcfe3b3517)
![모자이크1](https://github.com/Dongsun1234/BackgroundChromaKey/assets/130419965/30318a60-d5f6-4a4c-a834-c4f0a175d528)

```python
if key == 113:  # q 누르면 저장
	  suffix = datetime.datetime.now().strftime("%y%m%d_%H%M%S")  # 현재 시간으로 저장
	  cv.imwrite("/home/pi/Pictures/" + suffix + ".jpg", draw)  # 저장 경로 및 복사 이미지
#키보드 입력 'q'를 누르면 해당 폴더에 이미지 저장
```

## 조이스틱 파트
```python
import spidev

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

sw_val = readadc(sw_channel)
vrx_pos = readadc(vrx_channel)
vry_pos = readadc(vry_channel)
img = imagechange(vrx_pos, vry_pos, sw_val)

조이스틱 (x:0~1023 y:0~1023 중점 (510,150))을 이용하여 영역별 배경 이미지를 넣어주었고
조이스틱 스위치로 BGR색상을 랜덤으로 배경색을 변환하였고 마지막으로 중간 범위 (x:490~530,
y:490,530) 영역을 기준으로 배경 기본이미지를 넣어주었다.
```
