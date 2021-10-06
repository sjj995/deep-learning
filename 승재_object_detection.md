# 승재 - object_detection

### ■ 크롤링 데이터 : 구글에서 '전동 킥보드' 사진 100장 추출

```python
# 구글 웹스크롤링 코드 (훈련 데이터)
######################################################

import urllib.request # 웹 url을 파이썬이 인식 할 수 있게하는 패키지
from  bs4 import BeautifulSoup # html에서 데이터 검색을 용이하게 하는 패키지
from selenium import webdriver  
# selenium : 웹 애플리케이션의 테스트를 자동화하기 위한 프레임 워크 
# 손으로 클릭하면서 컴퓨터가 대신하면서 스크롤링하게 하는 패키지

from selenium.webdriver.common.keys import Keys
import time       # 중간중간 sleep 을 걸어야 해서 time 모듈 import

########################### url 받아오기 ###########################

# 웹브라우져로 크롬을 사용할거라서 크롬 드라이버를 다운받아 아래 파일경로의 위치에 둔다
# 팬텀 js로 하면 백그라운드로 실행할 수 있음
binary = 'c:\chromedriver/chromedriver.exe'

# 브라우져를 인스턴스화
browser = webdriver.Chrome(binary)

# 구글의 이미지 검색 url 받아옴(아무것도 안 쳤을때의 url) 
browser.get("https://www.google.co.kr/imghp?hl=ko&tab=wi&ei=l1AdWbegOcra8QXvtr-4Cw&ved=0EKouCBUoAQ") 

# 구글의 이미지 검색에 해당하는 input 창의 id 가 '  ?  ' 임(검색창에 해당하는 html코드를 찾아서 elem 사용하도록 설정)
# input창 찾는 방법은 원노트에 있음

# elem = browser.find_elements_by_class_name('gLFyf gsfi') # Tip : f12누른후 커서를 검색창에 올려두고 id를 찾으면 best

elem = browser.find_element_by_xpath("//*[@class='gLFyf gsfi']")  # 위의 코드대로 하거나 이렇게 하거나 둘 중 하나 select

########################### 검색어 입력 ###########################

# elem 이 input 창과 연결되어 스스로 햄버거를 검색
elem.send_keys("전동 킥보드") # 여기에 스크롤링하고싶은 검색어를 입력

# 웹에서의 submit 은 엔터의 역할을 함
elem.submit()

# 현재 결과 더보기는 구현 되어있지 않은상태 -> 구글의 경우 400개 image가 저장됨.

########################### 반복할 횟수 ###########################

# 스크롤을 내리려면 브라우져 이미지 검색결과 부분(바디부분)에 마우스 클릭 한번 하고 End키를 눌러야함

for i in range(1, 6): # 5번 스크롤 내려가게 구현된 상태 range(1,5)
    browser.find_element_by_xpath("//body").send_keys(Keys.END)
    time.sleep(10)          # END 키 누르고 내려가는데 시간이 걸려서 sleep 해줌 / 키보드 end키를 총 5번 누르는데 end1번누르고 10초 쉼

time.sleep(10)                      # 네트워크 느릴까봐 안정성 위해 sleep 해줌(이거 안하면 하얀색 이미지가 다운받아질 수 있음.)
html = browser.page_source         # 크롬브라우져에서 현재 불러온 소스 가져옴
soup = BeautifulSoup(html, "html.parser") # html 코드를 검색할 수 있도록 설정

browser.find_element_by_xpath("//*[@class='mye4qd']").click()  # 여기가 결과 더보기 코드입니다

for i in range(1, 5): # 4번 스크롤 내려가게 구현된 상태 range(1,5)
    browser.find_element_by_xpath("//body").send_keys(Keys.END)
    time.sleep(10)          # END 키 누르고 내려가는데 시간이 걸려서 sleep 해줌 / 키보드 end키를 총 5번 누르는데 end1번누르고 10초 쉼

time.sleep(10)                      # 네트워크 느릴까봐 안정성 위해 sleep 해줌(이거 안하면 하얀색 이미지가 다운받아질 수 있음.)
html = browser.page_source         # 크롬브라우져에서 현재 불러온 소스 가져옴
soup = BeautifulSoup(html, "html.parser") # html 코드를 검색할 수 있도록 설정

########################### 그림파일 저장 ###########################

### 검색한 구글 이미지의 url을 따오는 코드 ###
def fetch_list_url():
    params = []
    imgList = soup.find_all("img", class_="rg_i Q4LuWd")  # 구글 이미지 url 이 있는 img 태그의 _img 클래스에 가서 (f12로 확인가능.)
    for im in imgList:
        try :
            params.append(im["src"])                   # params 리스트에 image url 을 담음.
        except KeyError:
            params.append(im["data-src"])
    return params

# except부분
# 이미지의 상세 url 의 값이 있는 src 가 없을 경우
# data-src 로 가져오시오 ~  

def fetch_detail_url():
    params = fetch_list_url()

    for idx,p in enumerate(params,1):
        # 다운받을 폴더경로 입력
        urllib.request.urlretrieve(p, "c:/dataset/brand2/" + str(idx) + ".jpg")

# enumerate 는 리스트의 모든 요소를 인덱스와 쌍으로 추출
# 하는 함수 . 숫자 1은 인덱스를 1부터 시작해라 ~

fetch_detail_url()

# 끝나면 브라우져 닫기
browser.quit()
```

### ■ object_detection 코드

---

- classes : 1
- accuracy : 60.98%
- max batch : class * 2000
- filter : (4+1+classes)*3

```python
# 구글 드라이브 마운트
from google.colab import drive
drive.mount('/content/drive')

# 기존 디렉터리 하위경로 포함 삭제
import shutil
shutil.rmtree('/content/drive/MyDrive/yolo_custom_model_Training3')

# 그래픽 카드 확인
!nvidia-smi -L

# 새로운 디렉터리 생성
%cd /content/drive/MyDrive
!mkdir yolo_custom_model_Training3

# zip 파일 업로드 후 ls 명령어로 목록 확인
!ls '/content/drive/MyDrive/yolo_custom_model_Training3'

# 디렉터리 이동
%cd /content/drive/MyDrive/yolo_custom_model_Training3

# 이동 후 압축을 풀기 위한 디렉터리 생성
!mkdir custom_data

# 목록 확인
!ls '/content/drive/MyDrive/yolo_custom_model_Training3'

# 데이터 압축 풀기 (100장)
!unzip '/content/drive/MyDrive/yolo_custom_model_Training3/kickboard.zip' -d '/content/drive/MyDrive/yolo_custom_model_Training3/custom_data'

# 현재 경로 확인
%pwd

# darknet 다운로드
# download dataset in current directory(above path)
!git clone 'https://github.com/AlexeyAB/darknet.git' '/content/drive/MyDrive/yolo_custom_model_Training3/darknet'

# 디렉터리 이동
# move current directory to darknet
%cd /content/drive/MyDrive/yolo_custom_model_Training3/darknet

# Makefile 파일의 옵션들 변경
# change setting values in 'Makefile' file 
!sed -i 's/OPENCV=0/OPENCV=1/' Makefile
!sed -i 's/GPU=0/GPU=1/' Makefile
!sed -i 's/CUDNN=0/CUDNN=1/' Makefile
!sed -i 's/CUDNN_HALF=0/CUDNN_HALF=1/' Makefile
!sed -i 's/LIBSO=0/LIBSO=1/' Makefile

# 옵션을 변경한 모델 컴파일
# Compile model
"""  take care do not disconnect : file directory may be interupted 
if your network down during compile, I recommend delete darknet folder and restart number 4(get AlexeyAB/darknet)"""

!make

# 다크넷 셋팅 파일 경로로 이동
%cd ..
!darknet/darknet

# yolov4 사진 검출 모델 클론
!git clone 'https://github.com/jakkcoder/training_yolo_custom_object_detection_files' '/content/drive/MyDrive/yolo_custom_model_Training3/training_yolo_custom_object_detection_files-main'

%cd /content/drive/MyDrive/yolo_custom_model_Training3/training_yolo_custom_object_detection_files-main
# check out current dir files
!ls

# 이미지와 라벨링한 텍스트가 있는 디렉터리로 아래의 파이썬 코드 복사
# copy creating-train-and-test-txt-files.py & creating-files-data-and-name.py
"""creating-train-and-test-txt-files.py >> create 'train.txt' & 'test.txt' files
   creating-files-data-and-name.py >> create label 'labelled_data.data' file
   <if you excute both .py files, you get mentioned files upper lines 2,3>"""

!cp creating-train-and-test-txt-files.py /content/drive/MyDrive/yolo_custom_model_Training3//custom_data
!cp creating-files-data-and-name.py /content/drive/MyDrive/yolo_custom_model_Training3/custom_data

%cd /content/drive/MyDrive/yolo_custom_model_Training3/custom_data

# 파이썬 코드에서 옵션 변경
# change paths in both .py files
!sed -i '39 s@/home/my_name/Downloads/video-to-annotate@custom_data@' creating-train-and-test-txt-files.py
!sed -i '74 s@jpeg@jpg@' creating-train-and-test-txt-files.py
!sed -i '36 s@/home/my_name/Downloads/video-to-annotate@custom_data@' creating-files-data-and-name.py

# move current dir one step before
%cd ..

# creating-train-and-test-txt-files.py 실행하여 훈련데이터 테스트데이터 목록 생성
# excute .py file >> 'train.txt', 'test.txt'   
!python custom_data/creating-train-and-test-txt-files.py

# 라벨링 되어있는 정답데이터도 생성
# excute .py file >> 'labelled_data.data'
!python custom_data/creating-files-data-and-name.py

# 가중치 저장하기 위한 디렉터리 생성
# create directory 'custom_weight'
!mkdir custom_weight

%pwd

%ls -l

#yolov4.conv.137 모델 다운 받아 저장
# move 'yolov4.conv.137' file to 'custom_weight' dir
!mv yolov4.conv.137 custom_weight/

%cd /content/drive/MyDrive/yolo_custom_model_Training3/darknet/cfg

# 환경설정 하기 위해 따로 복사
# copy yolov4.cfg file & rename & paste
!cp yolov4.cfg yolov4_custom.cfg

# 복사한 파일로 클래스 및 배치사이즈 등 모델 설정
# change values for training
!sed -i '2 s@batch=64@batch=8@' yolov4_custom.cfg

!sed -i '7 s@width=608@width=416@' yolov4_custom.cfg
!sed -i '8 s@height=608@height=416@' yolov4_custom.cfg  

!sed -i '19 s@500500@2000@' yolov4_custom.cfg  #maxbatch=class*2000
!sed -i '21 s@400000,450000@1600,1800@' yolov4_custom.cfg  #maxbatch*0.8, maxbatch*0.9

!sed -i '968 s@classes=80@classes=1@' yolov4_custom.cfg
!sed -i '1056 s@classes=80@classes=1@' yolov4_custom.cfg
!sed -i '1144 s@classes=80@classes=1@' yolov4_custom.cfg

!sed -i '961 s@filters=255@filters=18@' yolov4_custom.cfg  #filters=(4+1+classes)*3 
!sed -i '1049 s@filters=255@filters=18@' yolov4_custom.cfg
!sed -i '1137 s@filters=255@filters=18@' yolov4_custom.cfg

%cd /content/drive/MyDrive/yolo_custom_model_Training3

# 가중치를 저장하기 위한 백업 파일 생성
!mkdir backup

%cd /content/drive/MyDrive/yolo_custom_model_Training3

# 다크넷 모델을 이용하여 학습 데이터들을 훈련
!darknet/darknet detector train custom_data/labelled_data.data darknet/cfg/yolov4_custom.cfg custom_weight/yolov4.conv.137 -dont_show

# 시각화 작업
# define helper function imShow
def imShow(path):
  import cv2
  import matplotlib.pyplot as plt
  %matplotlib inline

  image = cv2.imread(path)
  height, width = image.shape[:2]
  resized_image = cv2.resize(image,(3*width, 3*height), interpolation = cv2.INTER_CUBIC)

  fig = plt.gcf()
  fig.set_size_inches(18, 10)
  plt.axis("off")
  plt.imshow(cv2.cvtColor(resized_image, cv2.COLOR_BGR2RGB))
  #plt.show('')

%cd /content/drive/MyDrive/yolo_custom_model_Training3
!ls

# only works if the training does not get interrupted 
imShow('chart.png')

# 
%cd custom_data

!sed -i 's@custom_data@/content/drive/MyDrive/yolo_custom_model_Training3/custom_data@' test.txt
!sed -i 's@custom_data@/content/drive/MyDrive/yolo_custom_model_Training3/custom_data@' train.txt

!sed -i 's@custom_data@/content/drive/MyDrive/yolo_custom_model_Training3/custom_data@' labelled_data.data
!sed -i '5 s@.*@backup = /content/drive/MyDrive/yolo_custom_model_Training3/backup/@' labelled_data.data

!cat labelled_data.data

%cd /content/drive/My Drive/yolo_custom_model_Training3/darknet
!chmod +x ./darknet

# 클래스 정확도 확인
#You can check the mAP for all the saved weights to see which gives the best results

!./darknet detector map /content/drive/MyDrive/yolo_custom_model_Training3/custom_data/labelled_data.data /content/drive/MyDrive/yolo_custom_model_Training3/darknet/cfg/yolov4_custom.cfg /content/drive/MyDrive/yolo_custom_model_Training3/backup/yolov4_custom_final.weights -points 0

# 테스트 사진으로 테스팅
# run your custom detector with this command (upload an image to your google drive to test, the thresh flag sets the minimum accuracy required for object detection)

!./darknet detector test /content/drive/MyDrive/yolo_custom_model_Training3/custom_data/labelled_data.data /content/drive/MyDrive/yolo_custom_model_Training3/darknet/cfg/yolov4_custom.cfg /content/drive/MyDrive/yolo_custom_model_Training3/backup/yolov4_custom_final.weights /content/drive/MyDrive/yolo_custom_model_Training3/testing/kick1.jpg -thresh 0.3
imShow('predictions.jpg')

# run your custom detector with this command (upload an image to your google drive to test, the thresh flag sets the minimum accuracy required for object detection)

!./darknet detector test /content/drive/MyDrive/yolo_custom_model_Training3/custom_data/labelled_data.data /content/drive/MyDrive/yolo_custom_model_Training3/darknet/cfg/yolov4_custom.cfg /content/drive/MyDrive/yolo_custom_model_Training3/backup/yolov4_custom_final.weights /content/drive/MyDrive/yolo_custom_model_Training3/testing/kick2.png -thresh 0.3
imShow('predictions.jpg')

# run your custom detector with this command (upload an image to your google drive to test, the thresh flag sets the minimum accuracy required for object detection)

!./darknet detector test /content/drive/MyDrive/yolo_custom_model_Training3/custom_data/labelled_data.data /content/drive/MyDrive/yolo_custom_model_Training3/darknet/cfg/yolov4_custom.cfg /content/drive/MyDrive/yolo_custom_model_Training3/backup/yolov4_custom_final.weights /content/drive/MyDrive/yolo_custom_model_Training3/testing/kick3.jpg -thresh 0.3
imShow('predictions.jpg')

# run your custom detector with this command (upload an image to your google drive to test, the thresh flag sets the minimum accuracy required for object detection)

!./darknet detector test /content/drive/MyDrive/yolo_custom_model_Training3/custom_data/labelled_data.data /content/drive/MyDrive/yolo_custom_model_Training3/darknet/cfg/yolov4_custom.cfg /content/drive/MyDrive/yolo_custom_model_Training3/backup/yolov4_custom_final.weights /content/drive/MyDrive/yolo_custom_model_Training3/testing/kick4.jpg -thresh 0.3
imShow('predictions.jpg')
```

![res3.png](%E1%84%89%E1%85%B3%E1%86%BC%E1%84%8C%E1%85%A2%20-%20object_detection%2028386f10affd40089a26701e287436b3/res3.png)

![res1.png](%E1%84%89%E1%85%B3%E1%86%BC%E1%84%8C%E1%85%A2%20-%20object_detection%2028386f10affd40089a26701e287436b3/res1.png)

![res2.png](%E1%84%89%E1%85%B3%E1%86%BC%E1%84%8C%E1%85%A2%20-%20object_detection%2028386f10affd40089a26701e287436b3/res2.png)

![res4.png](%E1%84%89%E1%85%B3%E1%86%BC%E1%84%8C%E1%85%A2%20-%20object_detection%2028386f10affd40089a26701e287436b3/res4.png)