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


![mask_brown](https://github.com/Dongsun1234/BackgroundChromaKey/assets/130419965/9012f511-6a27-48fd-85ed-177a104b52ab)
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

