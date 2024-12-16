---
title: React에서 폼을 구현할 때의 디자인 패턴
date: 2024-12-12 16:30:00 +0800
categories: [ETC]
tags: [React, react-hook-form]
---

# React에서 폼을 구현할 때의 디자인 패턴

React에서 폼을 구현하려면 크게 두 가지 접근 방식이 있습니다. 바로 **Controlled Components**와 **Uncontrolled Components**입니다. 이번 글에서는 이 두 가지 방법을 비교하면서, React Hook Form이 왜 Uncontrolled Components를 채택했는지 살펴보겠습니다.

[React의 레거시 공식 문서](https://ko.legacy.reactjs.org/docs/uncontrolled-components.html)에서는 Controlled Components 기반의 폼 구현을 추천해왔고, Redux Form이나 Formik 같은 주요 폼 라이브러리도 Controlled Components를 사용하는 방식으로 설계되어 있습니다. 그런데 React Hook Form은 상대적으로 덜 주목받던 **Uncontrolled Components**를 채택했습니다. 그 이유가 뭘까요? 궁금하시죠? 바로 알아보겠습니다.

---

## Controlled Components

Controlled Components는 말 그대로 React State로 입력 값을 "컨트롤"하는 방식입니다. 사용자가 입력할 때마다 `onChange` 이벤트를 통해 State를 업데이트하고, 이 값을 다시 `value`로 렌더링합니다. 이런 방식은 State를 완벽히 제어할 수 있다는 장점이 있습니다. 하지만, 입력할 때마다 State가 업데이트되면서 리렌더링이 발생하기 때문에 폼이 복잡해질수록 성능 문제가 생길 수 있습니다.

### 코드 예제
```jsx
import React, { useState } from 'react';

const ControlledForm = () => {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [gender, setGender] = useState('male');

  const handleSubmit = (e) => {
    e.preventDefault();
    alert(JSON.stringify({ name, email, gender }));
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <select value={gender} onChange={(e) => setGender(e.target.value)}>
        <option value="male">남성</option>
        <option value="female">여성</option>
      </select>
      <button type="submit">제출</button>
    </form>
  );
};
```

---

## Uncontrolled Components

Uncontrolled Components는 입력 값이 React State가 아닌 DOM 자체(브라우저)에 의해 관리되는 방식입니다. React의 State를 사용하지 않기 때문에 입력할 때마다 리렌더링이 발생하지 않습니다. HTML 폼의 기본 동작을 그대로 활용하기 때문에 네이티브 HTML 방식과 비슷하다고 보시면 됩니다.

기본 값을 설정할 때는 `value` 대신 `defaultValue` 속성을 사용합니다.

### 코드 예제
```jsx
import React from 'react';

const UncontrolledForm = () => {
  const handleSubmit = (e) => {
    e.preventDefault();
    alert(JSON.stringify({
      name: e.target.name.value,
      email: e.target.email.value,
      gender: e.target.gender.value,
    }));
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" />
      <input name="email" />
      <select name="gender" defaultValue="male">
        <option value="male">남성</option>
        <option value="female">여성</option>
      </select>
      <button type="submit">제출</button>
    </form>
  );
};
```

### Uncontrolled Components의 장점

Uncontrolled Components의 대표적인 장점은 다음과 같습니다:

- **코드가 깔끔합니다.**: `onChange` 이벤트로 State를 업데이트하지 않아도 되기 때문에 불필요한 코드가 줄어듭니다.
- **성능이 좋습니다.**: 입력할 때마다 State가 변경되지 않아 리렌더링이 발생하지 않습니다.
- **HTML 폼에 가까운 구현 방식**: React 외의 환경(예: React Native)에서도 쉽게 적용할 수 있습니다.

하지만 Controlled Components가 필요한 경우도 있습니다:

- 입력 값을 자식 컴포넌트에 전달해야 할 때
- 입력 시 즉각적인 반응(자동완성 기능 등)이 필요할 때
- 복잡한 폼 상호작용(상호 의존적인 필드 등)이 필요한 경우

> 특정 필드만 Controlled Components처럼 동작하도록 설정할 수 있는 [useWatch](https://react-hook-form.com/docs/usewatch) 메서드를 제공합니다.

---

## React Hook Form

React Hook Form은 위에서 설명한 **Uncontrolled Components**의 장점을 최대한 활용하면서도, 간단하고 유연한 폼 검증을 제공하는 라이브러리입니다. 각 입력 필드의 DOM 참조를 `register`를 통해 관리하며, 이를 기반으로 유효성 검사를 처리합니다.

### 코드 예제
```jsx
import React from 'react';
import { useForm } from 'react-hook-form';

export default function App() {
  const { register, handleSubmit, formState: { errors } } = useForm();
  const onSubmit = (data) => alert(JSON.stringify(data));

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("name", { required: true })} />
      {errors.name && "이름은 필수 항목입니다"}

      <input {...register("email")} />
      <input {...register("age")} type="number" />

      <select {...register("gender")}>
        <option value="male">남성</option>
        <option value="female">여성</option>
      </select>

      <button type="submit">확인</button>
    </form>
  );
}
```

React Hook Form의 더 많은 사용법과 기능은 [공식 문서](https://react-hook-form.com/)에서 확인할 수 있습니다.

---

### React Hook Form의 장점

1. **성능**
  - Uncontrolled Components를 사용해 불필요한 리렌더링을 방지합니다.
  - `useForm`의 `mode` 옵션으로 유효성 검사 시에도 State 업데이트를 최소화합니다.
  - 특정 필드만 Controlled Components처럼 동작하도록 설정할 수 있는 [useWatch](https://react-hook-form.com/docs/usewatch) 메서드를 제공합니다.

2. **유효성 검사**
  - HTML5의 [Constraint Validation API](https://developer.mozilla.org/en-US/docs/Web/HTML/Constraint_validation)를 기본으로 지원합니다.
  - **Yup**, **Zod**, **Joi** 등 외부 검증 라이브러리와 쉽게 통합할 수 있는 [resolver](https://react-hook-form.com/docs/useform#resolver)를 제공합니다.

3. **경량성**
  - 외부 의존성이 없으며, 패키지 크기가 매우 작습니다.
  - `useForm` 훅만 사용할 경우 약 6.54KB로 매우 가볍습니다.

---

### 라이브러리 비교

| 라이브러리                                               | 문서화                  | 커뮤니티            | 업데이트         | 성능 중점 여부 | 비고                                               |
|-----------------------------------------------------|-------------------------|---------------------|------------------|----------------|--------------------------------------------------|
| [SurveyJS](https://surveyjs.io/)                    | 문서가 간단하고 따라하기 쉬움 | 커뮤니티가 크고 성장 중 | 업데이트가 자주 이루어짐 | 아니오         | 대규모 프로젝트에 유료 플랜 필요, 다양한 폼 기능 제공  |
| [React Hook Form](https://react-hook-form.com/)     | 문서가 간단하고 따라하기 쉬움 | 커뮤니티가 매우 크고 성장 중 | 업데이트가 자주 이루어짐 | 예            | React 기반 프로젝트에서 가장 인기 있는 폼 라이브러리 |
| [rc-field-form](https://github.com/react-component/field-form) | 문서가 조금 복잡하지만 충분히 활용 가능 | 커뮤니티가 작지만 성장 중 | 업데이트가 자주 이루어짐 | 예            | Ant Design 팀에서 개발, 안정적인 유지 관리 가능     |
| [Tanstack Form](https://tanstack.com/form/latest)   | 문서가 간단하고 따라하기 쉬움 | 커뮤니티가 크고 성장 중 | 업데이트가 매우 자주 이루어짐 | 예            | 현재 베타 단계, 안정화 전이라 API 변경 가능성 있음  |

---

