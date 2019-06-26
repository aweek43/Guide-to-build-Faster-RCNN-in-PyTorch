# Guide-to-build-Faster-RCNN-in-PyTorch-Korean
- https://medium.com/@fractaldle/guide-to-build-faster-rcnn-in-pytorch-95b10c273439
- 해당 사이트의 코드를 바탕으로 수정, 분석하였습니다.
- 해당 사이트 코드의 수많은 버그 해결했습니다.
- image size는 224로 수정했습니다.
- line by line으로 주석 설명 있습니다.
# 학습용 코드가 아닌 Faster R-CNN의 이해를 위한 코드입니다.
- 코드의 흐름은 RPN, NMS, Detector, Loss순입니다.
- 코드 요약입니다.
-
-1. 전체 image를 CNN에 넣음(classifier 제외) -> 1/16 downsampling 된 feature map 획득

-2. feature map의 한 픽셀당 크기, 비율 다른 9개 고정 크기 anchor 생성

-3. anchor이 원본 image에 벗어나지 않는 anchor만 따로 뽑음

-4. 각 anchor과 gt box와의 iou 구함

-5. 각 anchor마다 최대 iou를 가지는 gt box, 각 gt box마다 최대 iou를 가지는 anchor을 찾음

-6. 각 anchor마다 최대 iou가 0.3 미만이면 해당 anchor의 label은 0이다.
- 6-1. 각 anchor마다 최대 iou가 0.7 이상이면 해당 anchor의 label은 1이다.
- 6-2. 각 gt box마다 최대 iou를 가지는 anchor의 label은 1이다.
-(anchor label = 0: background, 1: object, -1: 애매하므로 무시)

-7. anchor label이 0인 것, 1인 것을 mini batch size의 반씩 뽑는다.

-8. anchor과 gt box와의 상대적인 크기를 구한다.

-9. 전체 anchor의 label과 상대 위치를 저장한다.

-10. feature map을 RPN classifier와 RPN regressors에 병렬적으로 넣는다.

-11. RPN regressors를 거친 예측 anchor들 중 image 범위 내의 anchor만 뽑는다.

-12. RPN classifier를 거친 예측 label과 예측 anchor을 매치시킨다.

-13. 예측 label이 높은 순서로 일정 개수만큼 뽑는다.

-14. 한 anchor을 기준으로 해당 anchor가 0.7 이상 겹치는 anchor을 지운다.(반복)

-15. 예측 anchor과 gt box와의 iou를 구한다.

-16. 각 예측 anchor마다 최대 iou를 가지는 gt box를 찾는다.

-17. 최대 iou가 0.7 이상인 것들 중에 랜덤하게 mini batch size * 0.25개 뽑는다.
- 17-1. 최대 iou가 0.5 이하인 것들 중에 랜덤하게 나머지 mini batch size만큼 뽑는다.

-18. 최대 iou가 0.5 이하는 예측 anchor은 label을 0(background)로 설정한다.

-19. 최종 선택된 예측 anchor(roi)과 gt box의 상대위치를 구한다.

-20. feature map에서 roi부분을 떼어내 pooling한다.

-21. pooling된 값을 classifier와 regressors에 병렬적으로 넣는다.

-22. Loss += RPN classifer와 gt box label과의 차이
- 22-1. Loss += (고정 크기 anchor과 gt box의 상대 위치)와 RPN regressors의 anchor 위치정보의 차이

-23. Loss += RoI classifier와 gt box label과의 차이
- 23-1. Loss += RoI regressors와 gt box와의 차이
