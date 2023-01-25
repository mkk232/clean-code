# clean-code
clean code for java with Udemy

## commit

- __항상 시작은 동사원형으로 시작해야함. (will이 생략되었다고 생각해야함)__
- __한개의 큰 작업에 한번의 commit을 하자.__
  - commit 자체도 SRP를 적용하고 지키자.
- __commit 메세지를 한글로 작성하는건 ? __
  - 국내와 국외의 개발자들과 협업하기 위해서는 영어로 하는게 좋다.
    - 본문의 내용은 영어로 할 필요까진 없지만 제목은 영어로 하는것을 권장.


## java
``` java
  public Optional<String> login(UserReqDto userReqDto) {
    Optional<User> byEmail = repository.findByEmail(userReqDto.getEmail());
                  // 사용자의 이메일을 받은것임. maybeUser 라고 하는것이 좋음.

    /*
      why ? 
      최대한 앞쪽에서 조건을 검색해서 더 검색할 필요가 없거나 오류를 반환하면 즉각 리턴해야함.
      그래야 뒤쪽에서 쓸데없는 일을 하지 않음.
      전체 소프트웨어의 속도를 떨어트릴 수 있음.
    */
    if(byEmail.isEmpty()) return Optional.empty(); // <-- 즉시 리턴 : 좋음 !

    
    User user = byEmail.get();
    
    if(!user.matchPassword(userReqDto.getPassword())) return Optional.empty(); // <-- 즉시 리턴 : 좋음 !
    

    AuthInfoDto authInfoDto = transform(user, AuthInfoDto.class); // <-- 이 위치를 변경하면 ? put 인수 위치와 일치하게..
    String sessionId = createSessionId(userReqDto);
    /*
      authInfoDto를 먼저 만들고 그 다음 sessionId를 받았음.
      만들어진 변수를 sessionStorage에 put 하여 넣을 때 인수를 생성한 순서에 맞게 적는것이 읽을 때 더 좋다.
    */
    sessionStorage.put(sessionId, authInfoDto);
    
    return Optional.ofNullable(sessionId);
  }
  
  public String createSessionId(UserReqDto userReqDto) {
    // TODO: session ID 암호화 // <-- TODO로 표기하는 방식 좋음
    /*
      현재 session ID가 암호화가 되어있지 않다는것을 알려주는 동시에 나중에 작업을 할 것이라고 알리기도 좋음.
      TODO를 이용해서 나중에 할 일을 정리하면 IDE가 TODO만 정리해서 알려주는 기능이 있음. 그래서 한번에 해야할 일을 모아서 볼 수 있음.
    */
    
    return userReqDto.getEmail();
  }
```
  
## Controller
``` java
  @GetMapping("/")
  public String index(Model model, @LoginUser SessionUser user) {
    if(user != null) {
      model.addAttribute("user", user);
      Map<Boolean, List<RaceListResponseDto>> raceMap = raceService.findByUserId(user.getId());
      model.addAttribute("ongoingRaceList", raceMap.get(Boolean.FALSE));
      model.addAttribute("completeRaceList", raceMap.get(Boolean.TRUE));
      // (-) of mustache(no counter to 'inverse selection' = verbose)
      if(raceMap.get(Boolean.TRUE) == null) model.addAttribute("completeRacesFlag", 0);
    } else {
      model.addAttribute("completeRacesFlag", 0);
    }
    return "index";
  }
  
  
  /*
    HTTP 동사로 처리하는게 좋다.
    이 메서드는 저장하는 매핑이니까 GetMapping이 아니라 저장하는 PostMapping이 좋다.
    GetMapping이니까 다른사람들은 스웨거와 같은 도구를 생각하게 된다.
    혼란을 방지하고자 GetMapping이 아니라 HTTP 동사에 맞는 어노테이션을 사용하자.
    end point에는 명사를 쓰고 HTTP 동사에는 동사를 쓰자.
  */
  @GetMapping("/race/save") // <-- RESTful API 저장/수정/삭제 등 행위를 HTTP verb로 처리 @PostMapping, @GetMapping, @PutMapping,
                              //               @DeleteMapping and @PatchMapping
  public String raceSave(Model model, @LoginUser SessionUser user) {
    model.addAttribute("fstUserId", user.getId());
    model.addAttribute("today", LocalDate.now());
    model.addAttribute("nextMonth", LocalDate.now().plusMonths(1));
    return "race-save";
  }
  
                                                                    // write니까 put이나 post가 되어야 함.
  @PreAuthorize("hasRole('ROLE_ADMIN') or hasPermission(#id, 'race', 'write')")
  @GetMapping("/race/update/{id}") // <-- PutMapping
                // raceUpdate의 명명을 HTTP 동사에 맞춰야 함.
  public String raceUpdate(Model model, @LoginUser SessionUser user. @PathVariable Long id) {
    RaceResponseDto race = raceService.findById(id);
    String userTitle = String.format("you (%s)", user.getName());
    
    model.addAttribute("race", race);
    boolean isFstUser = user.getId().equals(race.getFstUserId());
    model.addAttribute("fstUserTitle", isFstUser ? userTItle : your opponent");
    model.addAttribute("sndUserTitle", isFstUser ? "your opponent": userTitle);
    model.addAttribute(isFstUser ? "isFstUser": "isSndUser", 0);
    model.addAttribute("sndUserHabit", raceGetSndUserId() == null ? "TBD" : race.getSndUserHabit());
    
    return "race-update";
  }
```

## 부트스트래핑
- __다른 사람의 리뷰를 받을 수 없는 상황일때, 스스로 좋은 코드를 개발하는 방법은 무엇일까 ?__
  > 내 스스로 생각하기 보다 누군가에게 설명해주는것 처럼 문제를 객관화시켜서 제 3자의 입장에서 생각해보게 하는것이다. 또 다른 해결방법이 나올 수 있기 때문이다.


### QNA
- __주석은 상단에 적는게 좋나요? 아니면 오른쪽에 적는것이 좋나요 ?__ 
  > 상황에 따라 다르다.
  > 
  > 예를 들어 Class에 대한 전체 주석은 상단이다.
  > 
  > 메소드도 마찬가지로 상단에 적는것이 바람직하다.
  > 
  
  - __그럼 언제 오른쪽에 적는가 ?__
  
    > 너무 긴 줄이 아니면 인라인으로 적는것이 좋다.
    > 너무 길다면 상단에 적는것이 좋다
    > 상수 관련 해서도 그루핑할 땐 상단에 적는것이 좋다.

- __상단에서 maybe를 적는 이유는 ?__ 
  > Optional과 연관이 있다. maybe를 적으면 안봐도 Optional인 줄 안다.
