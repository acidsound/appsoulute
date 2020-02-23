---
templateKey: blog-post
title: '[찍먹] React Admin 은 쓸만한 물건인가?'
date: 2020-02-23T11:24:17.259Z
description: React Admin을 써보고 사용 가능한 도구인지 검증해본다.
tags:
  - react
  - frontend
  - administrator
  - admin
---
# web service 를 만들 때 마다 하는 일

사용자를 만들고 인증하고 메뉴를 구성하고 권한을 설정하고 등등의 일은 매번 반복되는 지루한 일이지만 이것 없이는 제대론 서비스라고 이야기하기 힘들다.

admin panel 같은 theme들은 많은데 디자인 뿐만이 아니라 논리적으로도 공통을 잘 뽑아 놓은 것이 있는지 궁금하긴 했다.

[https://marmelab.com/react-admin/](https://marmelab.com/react-admin/) 마침 이런 물건을 발견해서 적응 가능한지 학습 곡선이 어떤지 한번 시도해보기로 했고 이런 선별도 몇번 해보면 숙련되지 않을까하는 기대로 글을 적어보기로 한다.

개발환경은 평소에 애용하는 stackblitz로 시작한다.

codesandbox는 너무 무겁고 codepen은 좀 구식이다.

나의 선택은...

# Stackblitz에서부터 시작하기

React 프로젝트를 생성하고 https://marmelab.com/react-admin/Tutorial.html 부터 보자.

stackblitz가 기본 생성한 파일 중 Hello.js랑 style.css는 지우고

App.js 를 아래와 같이 만들고

```jsx
/* app.js */
import React from 'react'
export default ()=> <h1>Hello</h1>;
```

index.js 는

```jsx
/* index.js */
import React, { Component } from 'react';
import { render } from 'react-dom';
import App from './App';

render(<App />, document.getElementById('root'));

```

이와 같이 최소 구성 (안그래도 최소 구성이지만)으로 정리한다.

보통 이정도가 좋더라.

# JSON Provider부터 시작하기

React admin은 backend를 포함하지 않는다.

하지만 세상엔 수많은 backend로 사용할 수 있는 mockup과 API들이 가득하니 아무거나 가져와서 써도 된다.

주소만 쳐도 JSON을 반환해주는 https://jsonplaceholder.typicode.com는 나도 종종 애용하는데 React admin 저자도 그런가보다.

localhost에선 보통 http지만 stackblitz 같은 걸 사용할 땐 https이니 둘 다 적용될 수 있도록 //jsonplaceholder.typicode.com 를 데이터 프로바이더로 사용하자.

App.js 를 아래와 같이 수정한다.

```jsx
/* app.js */
import React from 'react';
import { Admin, Resource, ListGuesser } from 'react-admin';
import jsonServerProvider from 'ra-data-json-server';

const dataProvider = jsonServerProvider('//jsonplaceholder.typicode.com');

export default () => (
  <Admin dataProvider={dataProvider}>
      <Resource name="users" list={ListGuesser} />
  </Admin>
);
```

stackblitz는 package.json을 건들지 않아도 js 소스를 보고 유추하여 필요한 package를 설치해주는 점이 편하다.

일단 소스는 단순해서 좋다.

jsonServerProvider 객체를 dataProvider로 Admin component에게 제공하면 users라는 이름으로 //jsonplaceholder.typicode.com/users 를 접근하여 list를 가져오는 방식.

ListGuesser 가 name attribute를가지고 URL을 만들어 준다.

브라우저 개발자 도구에서 network를 보니

https://jsonplaceholder.typicode.com/users?_end=10&_order=ASC&_sort=id&_start=0

이와 같이 fetch를 보내 값을 가져온다.

일단 여기까지 github에 push하고 release를 하나 찍자.

콘솔을 보면 ListGuesser 라는 놈이 재밌는 짓을 하는데

```jsx
Guessed List:

export const UserList = props => (
    <List {...props}>
        <Datagrid rowClick="edit">
            <TextField source="id" />
            <TextField source="name" />
            <TextField source="username" />
            <EmailField source="email" />
            <TextField source="address.street" />
            <TextField source="phone" />
            <TextField source="website" />
            <TextField source="company.name" />
        </Datagrid>
    </List>
);
```

fetch해온 데이터를 근거로 List 아래 Datagrid를 뽑아준다.

Datagrid와 그 아래 항목에 대해 커스텀 할 수 있다는 소리.

# Column 선택하기

list에서 ListGuesser라는 걸 썼는데 이걸 UserList로 바꿀 수 있다.

users.js 에 UserList를 넣어서 만들어보자. console 창에 나온 것을 복사하여 만들자.

```jsx
/* users.js */
import React from 'react';
import { List, Datagrid, TextField, EmailField } from 'react-admin';

export const UserList = props => (
    <List {...props}>
        <Datagrid rowClick="edit">
            <TextField source="id" />
            <TextField source="name" />
            <EmailField source="email" />
            <TextField source="phone" />
            <TextField source="website" />
            <TextField source="company.name" />
        </Datagrid>
    </List>
);
```

username과 address.street 는 빼보았다.

남은 것은 users.js를 app.js에 적용하는 것

```jsx
import React from 'react';
import { Admin, Resource } from 'react-admin';
import { UserList } from './users'
import jsonServerProvider from 'ra-data-json-server';

const dataProvider = jsonServerProvider('//jsonplaceholder.typicode.com');

export default () => (
  <Admin dataProvider={dataProvider}>
      <Resource name="users" list={UserList} />
  </Admin>
);
```

잘 적용이 된다.

보니까 나머지 CRUD도 잘 될 것 같다. 

`<TextField source="website" />`를 `<UrlField source="website" />`로 바꾸는 것도 잘 된다. 정상적으로 링크도 되고 어짜피 attribute 만 맞추면 문제는 없겠다.

물론 `import { List, Datagrid, TextField, UrlField, EmailField } from 'react-admin';` 로 UrlField component를 추가해야한다.

이런 건 믿으면 될테고 JSON이 아닌 Data Provider랑 인증을 어떻게 풀어냈는지 너무 궁금하다.

# Admin Component 훑어보기

https://marmelab.com/react-admin/Admin.html 을 보니 원하는 최소조건인 dataProvider와 authProvider가 admin component에 있는 것을 발견했다.

dataProvider 는 지금은 prisma가 된 graphCool의 영향을 받은 것 같다.

```jsx
const dataProvider = {
    getList:    (resource, params) => Promise,
    getOne:     (resource, params) => Promise,
    getMany:    (resource, params) => Promise,
    getManyReference: (resource, params) => Promise,
    create:     (resource, params) => Promise,
    update:     (resource, params) => Promise,
    updateMany: (resource, params) => Promise,
    delete:     (resource, params) => Promise,
    deleteMany: (resource, params) => Promise,
}
```

getMany에서 조건절. 가령 특정 기간안에 속하는 것들을 가져온다던가 할때 어떻게 만들지 궁금하다.

https://jsonplaceholder.typicode.com/users?_end=10&_order=ASC&_sort=id&_start=0

아까 List에서 봤던 걸 보면 filter 조건을 위와 같이 준 것으로 보아 가능할 것 같긴 하다.

기본적으로 가지고 있는 속성 이외에도 extend 해서 사용가능해보인다 어짜피 params 를 받을 수 있으니.

결국 dataProvider랑 authProvider를 보아야하는데 다음에 authProvider를 먼저 봐야겠다.
