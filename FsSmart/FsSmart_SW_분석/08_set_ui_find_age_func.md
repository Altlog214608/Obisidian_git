

```python
self.ui_find_dlg.pb_age_teens.setStyleSheet(     
{False: dsStyle.pb_age_teens, True: dsStyle.pb_age_teens_selected}\    [self.check_find_age_range(dsNum.FIND_AGE_TEENS)] )
	
```
`

## 1. `{False: ..., True: ...}[some_boolean]` 의 의미

- `{False: dsStyle.pb_age_teens, True: dsStyle.pb_age_teens_selected}` 는 **딕셔너리**입니다.
    
    - 키(key)가 `False` 또는 `True`이고, 값(value)이 `dsStyle.pb_age_teens` 또는 `dsStyle.pb_age_teens_selected` 입니다.
        
- `[self.check_find_age_range(dsNum.FIND_AGE_TEENS)]` 는 이 딕셔너리에서 키를 조회하는 부분입니다.
    
- 즉, `self.check_find_age_range(dsNum.FIND_AGE_TEENS)` 함수가 `True` 또는 `False`를 반환하면,
    
    - 반환값이 `True`면 `dsStyle.pb_age_teens_selected`를 사용,
        
    - `False`면 `dsStyle.pb_age_teens` 스타일(원래 스타일)을 사용한다는 뜻입니다.
        

그래서, 전체는

- **버튼이 선택된 상태면 강조된 스타일(`pb_age_teens_selected`)을 씌우고, 아니면 기본 스타일(`pb_age_teens`)을 씌운다** 를 의미합니다.
    

## 2. 왜 `\` 가 있는가?

- 파이썬에서 한 줄이 너무 길 때 **줄바꿈을 하려면** `\`를 쓸 수 있습니다.
    
- 코드 가독성을 위해 `딕셔너리`와 `[인덱싱]`을 분리해서 다음 줄에 작성하고 싶을 때 이렇게 씁니다.
    
- 즉, 이 코드는 실제로는 아래처럼 한 줄과 동일합니다:
    

```python
self.ui_find_dlg.pb_age_teens.setStyleSheet(
{False: dsStyle.pb_age_teens, True: dsStyle.pb_age_teens_selected}[self.check_find_age_range(dsNum.FIND_AGE_TEENS)])

```

## 요약

- `check_find_age_range()` 함수가 `True`인지 `False`인지에 따라 두 스타일 중 선택해서 버튼에 동적으로 적용함
    
- `\`는 긴 코드를 보기 좋게 여러 줄로 나누기 위한 줄바꿈 문자임
    

