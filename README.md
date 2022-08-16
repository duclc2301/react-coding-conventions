## React

### Khoảng cách dòng

Đảm bảo có khoảng cách giữa import, tên component, state, các hàm xử lý, JSX,...

```tsx
import { useState } from 'react';
↕
const Counter = () => {
  const [count, setCount] = useState<number>(0);
  const [other, setOther] = useState<number>(0);
  ↕
  const handleClick = () => {
    setCount((count) => count + 1);
  };
  ↕
  return <div>{count}</div>;
};
↕
export default Counter;
```

### Export default

Tạo component bằng **Arrow function** và **export default** ở dưới cùng:

```tsx
const Component = () => {
  // ...
};

export default Component; ✅✅✅
```

Nếu cần export thêm các component khác, có thể viết gọn như sau:

```tsx
const Component = () => {
  // ...
};

export { Component as default, OtherComponent };
```

Hoặc

```tsx
const Component = () => {
  // ...
};

export default Component;
export { OtherComponent };
```

### Đặt tên

Đối với các trình xử lý sự kiện, sử dụng tiền tố **handle** theo sau là tên sự kiện:

```ts
const handleClick = () => {};

const handleChange = () => {};
```

Nếu trình xử lý sự kiện được truyền thông qua _props_, sử dụng tiền tố **on** theo sau là tên trình xử lý sự kiện:

```tsx
import Button from '@mui/material/Button';

interface Props {
  onClose: () => void;
}

const EnhancedButton = (props: Props) => {
  const { onClose } = props;

  return <Button onClick={onClose}>Click me!</Button>;
};

export default EnhancedButton;
```

Sử dụng:

```tsx
const Parent = () => {
  const handleClose = () => {
    // ...
  };

  return <EnhancedButton onClose={handleClose} />;
};
```

### Props

Khi cần sử dụng _props_, không destructuring trực tiếp trong tham số, thay vào đó destructuring ở _top-level_ của component:

```tsx
import Button from '@mui/material/Button';

interface Props {
  title: string;
  onClose: () => void;
}

const Component = (props: Props) => {
  const { title, onClose } = props; // ✅✅✅

  return <Button onClick={onClose}>{title}</Button>;
};

export default Component;
```

Nếu _props_ chứa _children_, cần sử dụng type **FCC** từ `types/react`:

```tsx
import Button from '@mui/material/Button';
import type { FCC } from 'types/react';

interface Props {
  onClose: () => void;
}

const Component: FCC<Props> = (props) => {
  const { children, onClose } = props;

  return <Button onClick={onClose}>{children}</Button>;
};

export default Component;
```

Trong trường hợp cụ thể hơn, chẳng hạn như các component bố cục, yêu cầu phải truyền vào hai _children_, lúc này chúng ta cần khai báo cụ thể cho _children_ sử dụng kiểu _Tuple_ như sau:

```tsx
import Box from '@mui/material/Box';
import type { ReactNode } from 'react';
import type { FCC } from 'types/react';

interface Props {
  children: [ReactNode, ReactNode];
}

const Component: FCC<Props> = (props) => {
  const { children } = props;

  return (
    <Box
      sx={{
        display: 'grid',
        gridTemplateRows: 'auto 1fr',
        height: 1,
      }}
    >
      {children}
    </Box>
  );
};

export default Component;
```

Xem thêm:

- [Tuple](https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types)

## Material UI

### Import

Luôn sử dụng import ở level 1:

```tsx
// Sử dụng
import Button from '@mui/material/Button'; // Level 1 ✅✅✅

// Thay vì
import { Button } from '@mui/material'; ❌❌❌
```

Khi import **type**, **interface**,..., phải sử dụng hậu tố _type_ sau import:

```tsx
import Button from '@mui/material/Button';
import type { ButtonProps } from '@mui/material/Button'; ✅✅✅

interface Props extends ButtonProps {
  // ...
}

const EnhancedButton = (props: Props) => {
  const { children, ...rest } = props;

  const handleClick = () => {
    // ...
  }

  return <Button onClick={handleClick} {...rest}>{children}</Button>;
};

export default EnhancedButton;
```

### Box

Không sử dụng các thẻ HTML gốc, component **Box** tương ứng với thẻ `</div>`, khi muốn sử dụng các thẻ HTML khác, sử dụng thuộc tính `component`, ví dụ:

```tsx {4}
import Box from '@mui/material/Box';

const Component = () => {
  return <Box component="span">Box is span</Box>;
};

export default Component;
// Kết quả: <span class="MuiBox-root mui-style-0">Box is span</span>
```

Xem thêm:

- [Box](https://mui.com/material-ui/react-box/)

### Container

Khi cần tạo container, sử dụng **Container** component, trong đó `maxWidth="lg"` sẽ tương ứng với **1200px**:

```tsx
import Container from '@mui/material/Container';

const Layout: FCC = (props) => {
  const { children } = props;

  return <Container maxWidth="lg">{children}</Container>;
};
```

Xem thêm:

- [Container](https://mui.com/material-ui/react-container/)
- [Breakpoints](https://mui.com/material-ui/customization/breakpoints/)

### Styles

Ưu tiên sử dụng **sx** để style cho component, trong trường hợp logic css quá phức tạp hoặc cần export để dùng chung thì mới sử dụng **styled**:

```tsx
import { styled } from '@mui/material/styles';

// Sử dụng sx ⚡
const Component = () => {
  return (
    <Box
      sx={{
        width: 1,
        bgcolor: 'background.default',
        color: active ? 'action.active' : 'action.hover',
      }}
    />
  );
};

// Viết bình thường
const Component = styled(Box)({
  width: '100%',
  height: '100%',
});

// Sử dụng theme
const Component = styled(Box)(({ theme }) => ({
  width: '100%',
  height: '100%',
  backgroundColor: theme.palette.background.default,
}));

// Sử dụng props
const Component = styled(Box, {
  shouldForwardProp: (prop: string) => !['active'].includes(prop),
})<{ active: boolean }>(({ theme, active }) => ({
  width: '100%',
  height: '100%',
  color: active ? theme.palette.action.active : theme.palette.action.hover,
}));

// Sử dụng props + spread syntax nếu cần áp dụng nhiều thuộc tính
const Component = styled(Box, {
  shouldForwardProp: (prop: string) => !['active'].includes(prop),
})<{ active: boolean }>(({ theme, active }) => ({
  width: '100%',
  height: '100%',
  color: theme.palette.action.hover,
  ...(active && {
    color: theme.palette.action.active,
    backgroundColor: theme.palette.background.default,
  }),
}));

// Cũng có thể sử dụng với các thẻ HTML (đã được áp dụng sẵn sx)
const Image = styled('img')({
  display: 'block',
  width: '100%',
  height: '100%',
});
```

Đối với **padding**, **margin**, sử dụng hàm `spacing()`, không sử dụng đơn vị trực tiếp:

```tsx
// Sử dụng styled
const Component = styled(Box)(({ theme }) => ({
  width: '100%',
  height: '100%',
  padding: theme.spacing(3), // 24px
}));

// Sử dụng sx
const Component = () => {
  return <Box sx={{ width: 1, height: 1, p: 3 }} />;
};
```

Xem thêm:

- [sx](https://mui.com/system/getting-started/the-sx-prop/)
- [styled](https://mui.com/system/styled/)
- [spacing](https://mui.com/system/spacing/)
- [style utilities](https://mui.com/system/properties/)

### CSS Rule

Mỗi MUI Component đều có các CSS Rule riêng để style, khi cần style một component thì phải sử dụng các rule này. Các rule đều được cung cấp thông qua một đối tượng theo quy ước `[name][Classes]`:

Ví dụ với **FormLabel** sẽ tương ứng với `formLabelClasses`:

```tsx
import type { FormLabelProps } from '@mui/material/FormLabel';
import FormLabel, { formLabelClasses } from '@mui/material/FormLabel';

interface LabelProps extends FormLabelProps {
  title: string;
  name: string;
  required?: boolean;
}

const EnhancedFormLabel = (props: LabelProps) => {
  const { title, name, required, children, ...rest } = props;

  return (
    <FormLabel
      sx={{
        [`& .${formLabelClasses.asterisk}`]: {
          color: 'error.light',
        },
      }}
      htmlFor={name}
      required={required}
      {...rest}
    >
      {children}
    </FormLabel>
  );
};

export default EnhancedFormLabel;
```

**Khi nào cần sử dụng phương pháp này và cần lưu ý điều gì?**

- Thông thường, khi sử dụng **sx**, các thuộc tính sẽ được áp dụng trực tiếp cho **root**, nếu điều này không đủ tính ưu tiên thì mới sử dụng đến các rule cụ thể hơn, chẳng hạn như **asterisk**.
- Lưu ý không viết trực tiếp các lớp như `.MuiFormLabel-asterisk`, thay vào đó cần sử dụng **formLabelClasses** và sau đó tham chiếu tới các css rule cụ thể, chẳng hạn `.${formLabelClasses.asterisk}`. Lý do cho điều này là khi build, các lớp css sẽ được hash, việc hardcode trực tiếp tên class có thể gặp lỗi.
- Đối với các lớp trạng thái như `.Mui-active`, `.Mui-checked`,... chúng sẽ không bao giờ thay đổi, do đó có thể sử dụng trực tiếp (nhưng nên sử dụng hạn chế).

Xem thêm:

- [State classes](https://mui.com/material-ui/customization/how-to-customize/#state-classes)

## TypeScript

### Any

Khai báo type rõ ràng ngay từ đầu, hạn chế sử dụng **any** nhiều nhất khi có thể.

### Event

Đối với các trình xử lý sự kiện, không viết trực tiếp vào JSX, thay vào đó sẽ tạo hàm ở trên và tham chiếu xuống ở dưới.

Các type cho các sự kiện phổ biến đều được cung cấp trong `types/react`, ví dụ:

```tsx
import TextField from '@mui/material/TextField';
import { useState } from 'react';
import type { ChangeEvent } from 'types/react'; // ✅✅✅

const Input = () => {
  const [value, setValue] = useState<string>('');

  const handleChange: ChangeEvent = (event) => {
    setValue(event.target.value);
  };

  return <TextField value={value} onChange={handleChange} name="name" />;
};

export default Input;
```
