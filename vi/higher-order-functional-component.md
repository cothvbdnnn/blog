# Higher Order Component với Reactjs

Ví dụ có một component để hiển thị ảnh và khi hover sẽ giảm opacity của ảnh. Đơn giản như sau: 

```javascript
import { useState } from 'react'

const ImageHover = ({ src, opacity }) => {
  const [isHovered, setIsHovered] = useState()
  return (
    <div
      style={{
        opacity: isHovered ? opacity : 1
      }}
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      <img src={src} alt="img" />
    </div>
  )
}

export default function Image() {
  return (
    <Image src="image" opacity={0.5} />
  )
}
```

Nhưng nếu chúng ta có một component khác và cũng muốn hover giống như component Image thì sao?

Nếu copy lại component Image, chúng ta bị lặp code rất nhiều. Vậy nên chúng ta sẽ sử dụng một số cách để có thể khắc phục điều này.

## Wrapper Components

Wrapper components là các thành phần bao quanh các components và cung cấp cấu trúc mặc định để hiển thị children components

Chúng ta sẽ tách ra như sau

Component Image sẽ chỉ nhận đầu vào là src và hiển thị ra ảnh
```javascript
// Image.jsx
const Image = ({ src }) => {
  return <img src={src} alt="img" />
}
```

Component HoverOpacity sẽ nhận `children` và `opacity`

```javascript
// HoverOpacity.jsx
import { useState } from 'react'

const HoverOpacity = ({ children, opacity }) => {
  const [isHovered, setIsHovered] = useState()
    return (
      <div
        style={{
          opacity: isHovered ? opacity : 1
        }}
        onMouseEnter={() => setIsHovered(true)}
        onMouseLeave={() => setIsHovered(false)}
      >
        { children }
      </div>
  )
}
```
```javascript
export default function WrappedComponent() {
  return (
    <div>
      <HoverOpacity opacity={0.5}>
        <Image src="image1" />
      </HoverOpacity>
      <HoverOpacity opacity={0.7}>
        <BackgroundImage src="image2" />
      </HoverOpacity>
    </div>
  )
}
```

## Higher Order Component

Higher order component là một component sẽ nhận đầu vào là một component và trả về một component khác

Vẫn sẽ tách ra một component Image với chức năng hiển thị ảnh khi nhận `src` và `alt`

```javascript
// Image.jsx
const Image = ({ src, alt = "img" }) => {
  return <img src={src} alt={alt} />
}
```

Và component HoverOpacity, nó sẽ nhận 

```javascript
// HoverOpacity
import { useState } from 'react'

const HoverOpacity = Component => function NewComponent(props) {
  const [isHovered, setIsHovered] = useState()
  const { opacity, alt, ...rest } = props
  const formatAlt = alt?.toLowerCase()?.replaceAll(' ', '-')
  return (
    <div
      style={{
        opacity: isHovered ? opacity : 1
      }}
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      {alt && <h1>{alt}</h1>}
      <Component {...rest} alt={formatAlt} />
    </div>
  )
}
```

```javascript
const HoverImage = HoverOpacity(Image)
const HoverBackgroundImage = HoverOpacity(BackgroundImage)

export default function WrappedComponent() {
  return (
    <div>
      <HoverImage src="img1" opacity={0.8} alt="Image 1" />
      <HoverBackgroundImage src="img2" opacity={0.5} />
    </div>
  )
}

```

```html
<img src="img1" alt="image-1">
```