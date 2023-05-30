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

# 하지만 몸 영역 외의 다른 배경에서도 같이 검출이 되는 것을 볼 수 있었다. 그리고 머리카락 영역에서도 완전히 검출되지 않았다.

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


![mask_mor_inv.jpg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7e7f9bb0-68b9-4670-954f-9feaedc1ab2e/mask_mor_inv.jpg)
