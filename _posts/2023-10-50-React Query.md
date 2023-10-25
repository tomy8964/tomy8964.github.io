---
title: React Query
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [프론트 , React]
tags:
  [
    프론트,
    React,
    React Query
  ]

toc: true
toc_sticky: true

---

# React-Query

## React-Query란?

- React 애플리케이션에서 데이터 가져오기 및 상태 관리에 사용되는 인기 있는 라이브러리
- API, 데이터베이스 또는 기타 소스에서 가져와야 하는 데이터를 관리
- React 구성 요소와 원활하게 통합하는 강력하고 유연한 방법을 제공

## 주요 기능

1. **데이터 fetching**: 캐싱, 자동 업데이트 및 백그라운드 패칭와 같은 데이터 패칭을 제공
2. **쿼리**: 쿼리는 외부 소스에서 데이터를 가져오고 관리하는 상태 관리 기능. React Query는 쿼리 결과를 메모리에 캐시하므로 빠르고 효율적인 데이터 검색이 가능하다. 또한 자동 백그라운드 데이터 업데이트 및 오래된 데이터 관리를 지원합니다.
3. **Muatation**: Mutation은 데이터 생성, 업데이트 또는 삭제와 같은 데이터 수정 작업을 처리하는 기능이다. React Query는 업데이트, 롤백 기능 등을 처리하는 변형 기능을 제공한다.
4. **캐싱**: React Query에는 데이터를 메모리에 저장하는 지능형 캐싱 메커니즘이 포함되어 있다. 캐시된 데이터는 후속 쿼리에 자동으로 사용되므로 중복 API 호출을 줄인다.
5. **invalidateQueries**: 라이브러리는 특정 쿼리를 유효하지 않은 것으로 표시하여 다시 액세스할 때 데이터를 다시 가져오라는 메시지를 표시할 수 있는 캐시 무효화를 지원한다. 이렇게 하면 데이터가 최신 상태로 유지된다.
6. **백그라운드 업데이트**: React Query는 일정한 간격으로 백그라운드에서 자동으로 데이터를 업데이트하여 애플리케이션의 데이터가 항상 최신 상태인지 확인할 수 있다.
7. **Pagination**: React Query는 pagination ****데이터 패칭 및 커서 기반 또는 오프셋 기반 pagination 관리를 위한 내장 라이브러리를 제공하여 pagination을 단순화한다.
8. **병렬 쿼리**: 여러 쿼리를 병렬로 가져오고 데이터가 변경될 때 모두 동시에 업데이트되도록 할 수 있다.
9. **글로벌 쿼리 구성**: 캐싱 기간, 재시도 동작 등과 같은 쿼리 및 변형에 대한 글로벌 구성을 설정할 수 있다.
10. **서버측 렌더링(SSR) 및 미리 가져오기**: React Query는 서버측 렌더링을 지원하고 페이지를 렌더링하기 전에 서버에서 데이터를 미리 가져올 수 있다.

## 사용 방법

### useQuery

- 데이터를 get 하기 위한 api
- 첫번째 파라미터로 unique Key가 들어가고, 두번째 파라미터로 비동기 함수(api호출 함수)가 들어간다.
- 첫번째 파라미터로 설정한 unique Key는 다른 컴포넌트에서도 해당 키를 사용하면 호출 가능하다.
- return 값은 api의 성공, 실패 여부, api return 값을 포함한 객체이다.
- useQuery는 `비동기`로 작동한다. **즉, 한 컴포넌트에 여러 개의 useQuery가 있다면 하나가 끝나고 다음 useQuery가 실행되는 것이 아닌 두 개의 useQuery가 동시에 병렬로 실행된다.** 여러 개의 비동기 query가 있다면 useQuery보다는 밑에 설명해 드릴 `useQueries`를 사용한다.

```javascript
const fetchProjects = async (): Promise<Project[]> => {
      try {
         const response = await api.get(`projects`);
         console.log(response.data);
         return response.data;
      } catch (error: any) {
         console.error('Error fetching projects', error);
         let status = error.code;
         if (error.response?.status != null) {
            status = error.response.status;
         }
         navigate(`../error?status=${status as string}`);
         return mockFetchProjectList();
      }
   };

   const projectListQuery = useQuery<Project[]>('projects', fetchProjects);
```

- `'projects'` 라는 키를 가진 `projectListQuery` useQuery 문이다.
- `fetchProjects` 라는 데이터를 가져오는 함수를 사용하고 있다.

```javascript
useEffect(() => {
      if (projectListQuery.isSuccess) {
         ...성공 후 로직
         setIsLoading(false);
      }
   }, [projectListQuery.isSuccess, isLoading]);
```

- `projectListQuery` 문의 성공 여부에 따라 다음 로직을 실행하면 된다.

### useMutation

- 값을 바꿀 떄 사용하는 api이다.
- return 값은 useQuery와 동일하다.

```javascript
const handleSubmitProject = async (event: React.FormEvent<HTMLFormElement>): Promise<void> => {
      event.preventDefault();

      const users: number[] = teamMembers.map(member => member.userId);
      const projectData = {
         projectName,
         description,
         users,
      };
      createProjectMutation.mutate(projectData);
   };

const createProjectMutation = useMutation(async (projectData: any) => await api.post('project', projectData), {
      onSuccess: async () => {
         setProjectName('');
         setDescription('');
         setTeamMembers([]);

         await queryClient.refetchQueries('projects').then(() => {
            console.log('refetch projects');
            navigate('../default');
         });
      },
      onError: error => {
         console.error('Error submitting project:', error);
         alert('서버 에러 입니다. 다시 시도하세요.');
      },
   });
```

- 여기 `createProjectMutation` 이라는 `post` 로 백엔드에 수정할 데이터를 보내는(프로젝트 정보를 수정하는) `useMutaion` 문이 있다.
- `onSuccess` 문이 성공하면 전에 다른 컴포넌트에서 사용한 `projects` 키를 사용한 `useQuery` 문을 `queryClient.refetchQueries('projects')` 을 통해 다시 패칭한다.