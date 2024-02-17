
- 카티시안 곱 = 교차 결합(Cross Join)

~~~ sql
SELECT * FROM EMPLOYEE, WORKS_ON;    %% 두 테이블은 교차 결합한다. %%

UPDATE EMPLOYEE, WORKS_ON SET salary = salary + 1000 WHERE (EMPLOYEE.id = WORKS_ON.empl_id and proj_id = 2003);
%% 두 테이블을 교차 결합하고 where 절 조건문으로 필터링을 수행한다. %%
~~~