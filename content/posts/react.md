---
description: "Первый useEffect используется единожды для того чтобы инициализировать IntersectionObserver и делать инкремент..."
title: "React + Typescript Infinite Scroll длиной в несколько строк"
date: 2021-06-11T23:34:25+05:00
draft: false
tags: ["react"]
---
### Будем использовать IntersectionObserver, более подробно можете ознакомиться здесь: https://developer.mozilla.org/ru/docs/Web/API/Intersection_Observer_API

***
```
import React, {useEffect, useRef, useState} from 'react'

type InfiniteScrollProps = {
  readonly loader: React.ReactElement,
  readonly hasMore: boolean,
  readonly next: (page: number) => unknown
}

export const InfiniteScroll: React.FC<InfiniteScrollProps> = ({
  loader,
  hasMore,
  next
}: InfiniteScrollProps): React.ReactElement => {
  const [page, setPage] = useState(1)
  const nextRef = useRef(next)
  const loaderRef: React.MutableRefObject<HTMLDivElement | null> = useRef(null)

  useEffect(() => {
    const observer = new IntersectionObserver(
      entities => entities[0].isIntersecting && setPage((page: number) => page + 1 )
    )

    observer.observe(loaderRef.current as HTMLDivElement)
  }, [])

  useEffect(() => {
    hasMore && nextRef.current(page)
  }, [page, hasMore])

  return <>{hasMore && <div ref={loaderRef}>{loader}</div>}</>

}
```
### Первый useEffect используется единожды для того чтобы инициализировать IntersectionObserver и делать инкремент текущей страницы в тот момент когда пользователь проскролит до элемента который мы передадим в метод observe()
### Второй useEffect будет вызывать функцию nextRef() если haseMore будет иметь положительное значение. В то же время nextRef() это та же функция next(), которую мы передаем в компонент, обернутая в useRef() для того чтобы не вызывать лишних вызовов useEffect()

***

### А вот пример использования с кастомным хуком

```
import React from 'react'
import {useDispatch, useSelector} from 'react-redux'
import {fetchProducts} from '@store/products/reducer'
import {selectAllProducts, selectHasMoreProducts} from '@store/products/selectors'
import {Products} from '@components/Products'
import {InfiniteScroll} from '@components/InfiniteScroll'

const useInfiniteScroll = () => {
  const dispatch = useDispatch()
  return {
    loader: <CircularProgress style={{marginTop: 50}} size={100} />,
    hasMore: useSelector(selectHasMoreProducts),
    next: (page: number) => dispatch(fetchProducts({page}))
  }
}

export const Home: React.FC = (): React.ReactElement => {
  const allProducts = useSelector(selectAllProducts)
  const infiniteScroll = useInfiniteScroll()
  return (
    <>
      <Products products={allProducts} />
      <InfiniteScroll {...infiniteScroll} />
    </>
  )
}
```
### Эта версия с минимальным базовым функционалом, прелесть в том что вы можете разобраться в данной реализации за пару минут дополнить ее под свои нужды



