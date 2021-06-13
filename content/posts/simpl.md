---
description: "Давайте представим что у нас есть самая обычная форма регистрации с полями..."
title: "Пишем валидаторы для Formik"
date: 2021-06-11T22:22:10+05:00
draft: false
tags: ["react"]

---

### Давайте представим что у нас есть самая обычная форма регистрации с полями firstName, lastName, email, password, confirmPassword. Все поля должны быть не пустыми, соответствовать определенной минимальной и максимальной длине, поле email должно подходить под регулярное выражение, password и confirmPassword должны совпадать.

![](https://miro.medium.com/max/882/1*pu4g9W0qh0j87ujf54ajAg.png)


```
import React from 'react';
import Button from '@material-ui/core/Button';
import CssBaseline from '@material-ui/core/CssBaseline';
import TextField from '@material-ui/core/TextField';
import Grid from '@material-ui/core/Grid';
import Typography from '@material-ui/core/Typography';
import { makeStyles } from '@material-ui/core/styles';
import Container from '@material-ui/core/Container';
import {Formik} from 'formik';

const useStyles = makeStyles((theme) => ({
  paper: {
    marginTop: theme.spacing(8),
    display: 'flex',
    flexDirection: 'column',
    alignItems: 'center',
  },
  avatar: {
    margin: theme.spacing(1),
    backgroundColor: theme.palette.secondary.main,
  },
  form: {
    width: '100%', // Fix IE 11 issue.
    marginTop: theme.spacing(3),
  },
  submit: {
    margin: theme.spacing(3, 0, 2),
  },
}));

export default function SignUp() {
  const classes = useStyles();

  return (
    <Container component="main" maxWidth="xs">
      <CssBaseline />
      <div className={classes.paper}>
        <Typography component="h1" variant="h5">
          Sign up
        </Typography>
        <Formik
          initialValues={{ 
            firstName: '', 
            lastName: '', 
            email: '', 
            password: '', 
            confirmPassword: '' 
          }}
          validate={() => console.log('need to add validation')}
          onSubmit={(values, { setSubmitting }) => {
            setTimeout(() => {
              alert(JSON.stringify(values, null, 2));
              setSubmitting(false);
            }, 400);
          }}
        >
          {({
            values,
            errors,
            touched,
            handleChange,
            handleBlur
          }) => (
            <form className={classes.form} noValidate>
              <Grid container spacing={2}>
                <Grid item xs={12} sm={6}>
                  <TextField
                    autoComplete="fname"
                    name="firstName"
                    variant="outlined"
                    required
                    fullWidth
                    id="firstName"
                    label="First Name"
                    autoFocus
                    onChange={handleChange}
                    onBlur={handleBlur}
                    value={values.firstName}
                    helperText={errors.firstName && touched.firstName && errors.firstName}
                    error={!!errors.firstName}
                  />
                </Grid>
                <Grid item xs={12} sm={6}>
                  <TextField
                    variant="outlined"
                    required
                    fullWidth
                    id="lastName"
                    label="Last Name"
                    name="lastName"
                    autoComplete="lname"
                    onChange={handleChange}
                    onBlur={handleBlur}
                    value={values.lastName}
                    helperText={errors.lastName && touched.lastName && errors.lastName}
                    error={!!errors.lastName}
                  />
                </Grid>
                <Grid item xs={12}>
                  <TextField
                    variant="outlined"
                    required
                    fullWidth
                    id="email"
                    label="Email Address"
                    name="email"
                    autoComplete="email"
                    onChange={handleChange}
                    onBlur={handleBlur}
                    value={values.email}
                    helperText={errors.email && touched.email && errors.email}
                    error={!!errors.email}
                  />
                </Grid>
                <Grid item xs={12}>
                  <TextField
                    variant="outlined"
                    required
                    fullWidth
                    name="password"
                    label="Password"
                    type="password"
                    id="password"
                    autoComplete="current-password"
                    onChange={handleChange}
                    onBlur={handleBlur}
                    value={values.password}
                    helperText={errors.password && touched.password && errors.password}
                    error={!!errors.password}
                  />
                </Grid>
                <Grid item xs={12}>
                  <TextField
                    variant="outlined"
                    required
                    fullWidth
                    name="confirmPassword"
                    label="Confirm Password"
                    type="password"
                    id="confirm-password"
                    autoComplete="current-password"
                    onChange={handleChange}
                    onBlur={handleBlur}
                    value={values.confirmPassword}
                    helperText={errors.confirmPassword && touched.confirmPassword && errors.confirmPassword}
                    error={!!errors.confirmPassword}
                  />
                </Grid>
              </Grid>
              <Button
                type="submit"
                fullWidth
                variant="contained"
                color="primary"
                className={classes.submit}
              >
                Sign Up
              </Button>
            </form>
          )}
        </Formik>
      </div>
    </Container>
  );
}
```


***

### Для начала напишем типы для будущей реализации
     
```
export type FormValue = {
  readonly [key: string]: string
}

export type ValidationErrors = {readonly [key: string]: string}

export type Validate = (values: FormValue) => ValidationErrors

export type ValidateFn = (form: FormValue, fieldName: string) => ValidationErrors

export type ValidationSchema = {readonly [key: string]: readonly ValidateFn[]};
```
### FormValue — это значение формы которое мы будем передавать в валидаторы
### ValidationErrors — ошибки валидации
### Validate — тип той самой функции которую мы будем передавать в Formik
### ValidationSchema — вернемся к ней чуточку позже
### Ниже приведен пример реализации валидатора min, который будет проверять что значение больше определенной длины

```
export const min =
  (target: number) => 
  (form: FormValue, fieldName: string) => 
    (form[fieldName] as string)?.length < target
    ? {[fieldName]: `Should be more than ${target - 1} characters`} : {}
```
 * min — это унарная функция которая принимает минимальное число символов и возвращает бинарную функцию которая принимает значение формы и название валидируемого поля, а возвращает либо ошибку либо пустой объект

### Для удобства воспользуемся функцией curry которая преобразует функцию от нескольких аргументов к набору функций с одним аргументом.

```
export const curry = (fn, ...par) => {
  const curried = (...args) => (
    fn.length > args.length
      ? curry(fn.bind(null, ...args))
      : fn(...args)
  )

  return par.length
    ? curried(...par)
    : curried
```
У меня проблемы с тем как типизировать эту функцию на typescript, напишите свои предложения в комментариях

### Функция curry рекурсивная и понять как она работает довольно сложно, суть ее работы заключается в том что функцию которую мы с помощью нее создаем можно будет вызывать не сразу со всеми аргументами, а передавать их постепенно, вычисление произойдет только тогда, когда будут переданы все аргументы. Более подробно можете ознакомиться здесь: https://github.com/HowProgrammingWorks/PartialApplication

```
export const min = curry(
  (
    target: number, 
    form: FormValue, 
    fieldName: string
  ) => (form[fieldName] as string)?.length < target
    ? {[fieldName]: `Should be more than ${target - 1} characters`} : {}
)
```
Мы просто оборачиваем функцию от трех аргументов в curry, таким образом получаем каррированую функцию min

### То же самое с валидатором max

```
export const max = curry(
  (
    target: number, 
    form: FormValue, 
    fieldName: string
  ) => (form[fieldName] as string).length > target
    ? {[fieldName]: `Should be less than ${target} characters`} : {}
)
```

### Валидатор required это результат вызова валидатора min с минимальным числом символов равным единице

```
export const required = curry(
(form: FormValue, fieldName: string) => 
  Object.keys(
    min(1, form, fieldName)
  ).length 
    ? {[fieldName]: 'Required'} 
    : {}
)
```
### В валидаторе email мы просто проверяем на соответствие регулярному выражению

```
export const email = curry(
  (form: FormValue, fieldName: string) =>
    !(form[fieldName] as string).match(/\S+@\S+\.\S+/)
      ? {[fieldName]: 'Invalid email'} : {}
)
```

### В валидатор compare мы передаем ключ поля значение которого должно совпадать с тем что мы валидируем

```
export const compare = curry(
  (targetFieldName: string, form: FormValue, fieldName: string) =>
    form[fieldName] !== form[targetFieldName]
      ? {[fieldName]: 'Not equal'} : {}
)
```
***

### Валидаторы готовы, теперь нужно написать функцию getErrors которая будет принимать в себя данные вместе с массивом валидаторов и возвращать объект ошибки для нужного поля

```

export const getErrors = curry(
  (
    form: FormValue,
    fieldName: string,
    validators: readonly ValidateFn[]
  ): ValidationErrors =>
    validators
      .map(validator => validator(form, fieldName))
      .reduce((prev: ValidationErrors, curr: ValidationErrors) => ({
        ...prev,
        ...curr
      }), {})
)
```
В цикле map проходим по каждому элементу массива валидаторов и возвращаем объекты ошибки, получим массив объектов ошибки, далее с помощью функции reduce скомпонуем все ошибки в один объект

### Теперь создадим функцию validate которую будем передавать в Formik

```
const validate: Validate = (form: FormValue) => ({
  ...getErrors(form, 'firstName', [required, min(2), max(20)]),
  ...getErrors(form, 'lastName', [required, min(2), max(20)]),
  ...getErrors(form, 'email', [required, email]),
  ...getErrors(form, 'password', [required, min(6), max(20)]),
  ...getErrors(form, 'confirmPassword', [required, min(6), max(20), compare('password')])
})
```
### Слишком много кода, не правда ли? Каждый раз передаем один и тот же параметр form. Так как getErrors каррирована мы можем частично применить ее и сохранить form внутри замыкания

```
const validate: Validate = (form: FormValue) => {
  const getFormErrors = getErrors(form)

  return {
    ...getFormErrors('firstName', [required, min(2), max(20)]),
    ...getFormErrors('lastName', [required, min(2), max(20)]),
    ...getFormErrors('email', [required, email]),
    ...getFormErrors('password', [required, min(6), max(20)]),
    ...getFormErrors('confirmPassword', [required, min(6), max(20), compare('password')])
  }
}
```
### Все равно такое решение едва ли можно назвать удобным, предлагаю написать функцию getValidateFunction которая будет принимать объект такого вида:

```
{
  firstName: [required, min(2), max(20)],
  lastName: [required, min(2), max(20)],
  email: [required, email],
  password: [required, min(6), max(20)],
  confirmPassword: [required, min(6), max(20), compare('password')]
}
```
### И возвращать функцию validation. Вот пример реализации:

```
export const getValidateFunction =
  (validationSchema: ValidationSchema) => 
    (form: FormValue) =>
      Object.fromEntries(
        Object.entries(validationSchema).map(
          ([fieldName, value]) => [
            fieldName,
            getErrors(form, fieldName, value)[fieldName]
          ]
        )
      )
```

### Мы таким образом спрятали вызовы функции getErrors под капотом и получили чистый и удобный api

```
const validate: Validate = getValidateFunction(
  {
    firstName: [required, min(2), max(20)],
    lastName: [required, min(2), max(20)],
    email: [required, email],
    password: [required, min(6), max(20)],
    confirmPassword: [required, min(6), max(20), compare('password')]
  }
)
```
### Далее просто передаем validate в одноименный параметр компонента Formik

```

const validate: Validate = getValidateFunction(
  {
    firstName: [required, min(2), max(20)],
    lastName: [required, min(2), max(20)],
    email: [required, email],
    password: [required, min(6), max(20)],
    confirmPassword: [required, min(6), max(20), compare('password')]
  }
)

const initialValues = { 
  firstName: '', 
  lastName: '', 
  email: '', 
  password: '', 
  confirmPassword: '' 
}

/////////////////////////////////////////////

<Formik
  initialValues={initialValues}
  validate={validate} // put validate here
>
  {
    ({
      values,
      errors,
      touched,
      handleChange,
      handleBlur
    }) => (...) // form here
  }
</Formik>
```
# И все работает!

![](https://miro.medium.com/max/2400/1*3Upbty9ZnRz32dybjeDVDw.png)

### Исходный код можете найти в репозитории:
https://github.com/RYAZHAPOVILNUR/formik-validators

