# 전자사랑모임 설계내용 정리(react-native)

## 전자사랑모임: 모 대기업의 퇴직임원들의 커뮤니티 형성을 위해 만들어진 모임의 모바일 앱

[전자사랑모임 - Apps on Google Play](https://play.google.com/store/apps/details?id=com.eclubapp)

# 담당영역

### (2020.12~2021.08)

### 로그인, 회원가입

- 로그인, 회원가입 ui 설계
- redux, 액션함수, reducer 유지보수
- 기타 기능 설계
    - 인증번호 발송요청 후 타이머 설정
    - 비밀번호 재설정시 기존 사용 비밀번호와 일치여부 체크
- ui 이미지

<img width="362" alt="스크린샷 2021-10-30 오후 6 14 15" src="https://user-images.githubusercontent.com/73742422/139528485-742faefa-337a-4774-ae21-9d1d3f2affd2.png">


<img width="363" alt="스크린샷 2021-10-30 오후 6 15 33" src="https://user-images.githubusercontent.com/73742422/139528489-8a9c75ec-9171-4bad-a01a-8bbfbc058346.png">


<img width="362" alt="스크린샷 2021-10-30 오후 6 18 32" src="https://user-images.githubusercontent.com/73742422/139528505-c36d9532-137d-457e-9037-41cd8637ef87.png">

<img width="362" alt="스크린샷 2021-10-30 오후 6 17 53" src="https://user-images.githubusercontent.com/73742422/139528509-aeb04a1b-1578-46f0-bafc-5ec40cc4a739.png">


### 마이페이지

- redux, 액션함수, reducer 유지 보수
- 화상회의
    - 개설된 화상회의 목록 요청
    - 화상회의 개설
    - 화상회의 정보 확인 후 화상회의 앱 연결([하이퍼미팅(TmaxOS)](https://play.google.com/store/apps/details?id=com.hypermeeting))
    
    ```jsx
    // 화상회의 접근확인
        const accessMeetingConfirm = (videoData) => {
            console.log('access Meeting: ', videoData);
            setSelectedData(videoData);
            if(videoData.roomState === 'STATE_OPENED') {
                if(videoData.hasPassword) {
                    setPwdModal(true);
                }
                else accessMeeting(videoData);
            }
            else {
                makeDialog_OK(
                    '',
                    '회의가 전체 종료되었습니다.',
                    () => {}
                );
            }
        }
        // 화상회의 접근
        const accessMeeting = async (videoData) => {
            setPwdModal(false);
            setInviteCheck(true);
            try {
    						// 화상회의 정보 요청
                const result = await dispatch(
                    getHMeeting({
                        subPath: `/${videoData.roomId}`,
                        params: null,
                        data: null,
                    })
                );
    
                console.log('Video accessMeeting result: ', result);
                setInviteCheck(false);
                if(result.data.roomState === 'STATE_OPENED') {
                    const usersArr = result.data.hypermeetingUsers;
                    let canAccess = false
    								// 로그인된 사용자가 회의에 초대된 경우
                    if(usersArr) {
                        usersArr.forEach(function(user) {
                            if(user.id.userId === userId) canAccess = true;
                        })
                    }
    								// 회의 개설자인 경우
                    if(result.data.createdBy === userId) canAccess = true;
    								// 관리자인 경우
                    if(userId === 'admin') canAccess = true;
    
                    if(canAccess) {
    										// 화상회의 접근
                        RunHyperMeeting({
                            userId,
                            name: myInfo.name,
                            roomId: result.data.roomId,
                            password: result.data.roomPassword,
                            leaderKey: result.data.leaderKey,
                            isLeader: (result.data.createdBy === userId)
                        });
                    }
                    else {
                        makeDialog_OK(
                            '',
                            '접근 권한이 없습니다.',
                            () => {}
                        );
                    }
                }
                else {
                    makeDialog_OK(
                        '',
                        '회의가 전체 종료되었습니다.',
                        () => {}
                    );
                }
            }
            catch(e) {
                console.log('Video accessMeeting error: ', e);
                Toast.show('화상회의 정보요청 오류발생! 관리자에게 문의해주세요.');
            }
        }
    
    const RunHyperMeeting = async ({ userId, name, roomId, password, leaderKey, isLeader }) => {
    
      console.log('runHyperMeeting', userId, name, roomId, password, leaderKey);
    
    	// 하이퍼미팅 측 화상회의 개설 여부 요청
      await axios({
        method:'get',
        url:`https://hypermeeting.biz/restapi/v1.1/hmRoom/room/${roomId}`
      })
      .then(async response=>{
        console.log('response in isLeader, ', response);
        const isLeaderJoined=response.data.leaderJoined;
        
    		// 링크
        const hyper_meeting_url =
        (isLeader&&!isLeaderJoined)? `tmax://hypermeeting.biz/${roomId}?userId=${userId}&userName=${name}&password=${password}&leaderKey=${leaderKey}`
        : `tmax://hypermeeting.biz/${roomId}?userId=${userId}&userName=${name}&password=${password}`;
      const install_url =
        Platform.OS === 'android' ? 'https://play.google.com/store/apps/details?id=com.hypermeeting'
        // : 'https://apps.apple.com/ca/app/%ED%95%98%EC%9D%B4%ED%8D%BC%EB%AF%B8%ED%8C%85-%EB%82%B4-%EC%86%90%EC%95%88%EC%9D%98-%ED%9A%8C%EC%9D%98%EC%8B%A4/id1542751569';
        : encodeURI('https://apps.apple.com/ca/app/하이퍼미팅-내-손안의-회의실/id1542751569')
    
        console.log('hypermeeting url, in RunHyperMeeting, ', hyper_meeting_url, isLeader);
      
      const supported = await Linking.canOpenURL(hyper_meeting_url);
    	
    	// 하이퍼미팅 앱이 설치된 경우
      if (supported) {
        await Linking.openURL(hyper_meeting_url); 
      }
    	// 설치가 안된 경우 
    	else {
        await Linking.openURL(install_url);
      }
    
      })
      .catch(e=>{
        Toast.show('회의실 정보를 받아오는 데 실패하였습니다.');
        console.log('error in isLeader in RunHyperMeeting, ', e);
        result=false;
      })
      .finally(()=>{
        return result;
      })
    
      
    
    }
    ```
    
    <img width="361" alt="스크린샷 2021-10-30 오후 6 22 08" src="https://user-images.githubusercontent.com/73742422/139528523-3f393bc1-4d2c-41c0-9bcb-f79747a11daa.png">

    
    <img width="363" alt="스크린샷 2021-10-30 오후 6 22 33" src="https://user-images.githubusercontent.com/73742422/139528535-213263cf-1181-4ba2-8096-7044308c74c4.png">
    

### 모임등록

- 분과, 동호회 별 모임 등록 페이지 ui 설계

<img width="362" alt="스크린샷 2021-10-30 오후 6 32 23" src="https://user-images.githubusercontent.com/73742422/139528539-2353ab9e-daba-4972-ae8f-c12e09b27835.png">

### (2021.08~현재)

### 전자사랑 전 영역 유지보수 업무
