---
title: 'Building Reusable Components with TypeScript and React'
description: "In this blog post, we'll explore how to build reusable and type-safe components using TypeScript in React. We'll walk through creating a simple Button component, explaining the benefits of TypeScript's type safety, and demonstrating how to use the component effectively in your application."
publishedAt: 2025-03-11
tags: ['typescript', 'react']
---

#### Creating reusable components is one of the core principles of building scalable and maintainable front-end applications. With TypeScript, we can enhance this process by providing type safety and improved developer experience.

## Why TypeScript?

When building React applications at scale, plain JavaScript can quickly turn into a guessing game — what props does this component accept? What shape does this object have? TypeScript removes the guesswork by catching these questions at compile time instead of runtime.

For reusable components specifically, TypeScript gives us:

- **Self-documenting APIs** — your component's props are the documentation
- **Autocomplete everywhere** — editors can suggest valid props and values as you type
- **Safer refactors** — rename a prop and the compiler tells you everywhere it breaks
- **Fewer runtime surprises** — catch type mismatches before they ever ship

Let's put this into practice by building something every app needs: a Button component.

## Building a Type-Safe Button Component

Let's start with the basics — a Button component that wraps the native `<button>` element while adding our own styling variants:

```tsx
import { ButtonHTMLAttributes, ReactNode } from 'react';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  children: ReactNode;
}

export function Button({
  variant = 'primary',
  size = 'md',
  children,
  className,
  ...rest
}: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant} btn-${size} ${className ?? ''}`}
      {...rest}
    >
      {children}
    </button>
  );
}
```

A few things worth calling out here:

1. **Extending native attributes** — by extending `ButtonHTMLAttributes<HTMLButtonElement>`, our component automatically supports `onClick`, `disabled`, `type`, and every other native button prop, without us having to redeclare them.
2. **Union types for variants** — `'primary' | 'secondary' | 'ghost'` restricts the `variant` prop to a known set of values. Typo `'primarry'` and TypeScript stops you before your CSS does.
3. **Sensible defaults** — `variant = 'primary'` and `size = 'md'` mean consumers only need to specify what's different from the default.

Using it is exactly as predictable as it looks:

```tsx
<Button variant='primary' onClick={() => console.log('clicked')}>
  Save changes
</Button>

<Button variant='ghost' size='sm' disabled>
  Cancel
</Button>
```

Try passing `variant='danger'` and your editor will immediately flag it — long before a user ever sees a broken button.

## Making Components Generic

Buttons are simple, but real applications are full of components that need to work with different data shapes — lists, tables, dropdowns, selects. This is where generics shine.

Take a `List` component that should render any kind of item:

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => ReactNode;
  keyExtractor: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}
```

Now `List` can render anything — users, products, notifications — while still giving you full type safety on `item` inside `renderItem`:

```tsx
interface User {
  id: number;
  name: string;
  email: string;
}

<List<User>
  items={users}
  keyExtractor={(user) => user.id}
  renderItem={(user) => (
    <span>
      {user.name} — {user.email}
    </span>
  )}
/>;
```

Notice that inside `renderItem`, `user` is fully typed as `User` — TypeScript infers it from the `items` array, so there's no manual casting and no `any` in sight.

## Polymorphic Components with `as`

Sometimes a component's underlying element needs to change depending on context — think of a `Text` component that should render as a `<p>`, `<span>`, or `<label>` depending on where it's used. This is where things get a little more advanced:

```tsx
type TextProps<T extends React.ElementType> = {
  as?: T;
  children: ReactNode;
} & Omit<React.ComponentPropsWithoutRef<T>, 'as' | 'children'>;

function Text<T extends React.ElementType = 'p'>({
  as,
  children,
  ...rest
}: TextProps<T>) {
  const Component = as || 'p';
  return <Component {...rest}>{children}</Component>;
}
```

This pattern lets consumers swap the rendered element without losing type checking on its props:

```tsx
<Text>Default paragraph text</Text>

<Text as='label' htmlFor='email'>
  Email address
</Text>

<Text as='a' href='/about'>
  Learn more
</Text>
```

Pass `as='a'` and TypeScript knows you now need an `href`. Pass `as='label'` and it expects `htmlFor`. That's the kind of contextual type-checking that's nearly impossible to replicate in plain JavaScript.

## Best Practices for Reusable Components

A few principles that have served me well when designing component APIs:

1. **Compose, don't configure** — instead of stacking ten boolean props (`hasIcon`, `isRounded`, `withShadow`...), prefer composition with `children` and slots. Fewer props means fewer combinations to test and document.

2. **Keep prop types narrow** — prefer `'sm' | 'md' | 'lg'` over `string`. The narrower the type, the more the compiler can do for you.

3. **Export your prop types** — if `ButtonProps` is exported, consumers can extend it, write wrapper components, or build their own type-safe variants on top of yours:

```tsx
export type { ButtonProps };
```

4. **Avoid `any` at all costs** — it's tempting when a type gets complicated, but `any` silently disables type checking for everything it touches. `unknown` plus a type guard is almost always the better escape hatch.

5. **Document with JSDoc** — TypeScript-aware editors surface JSDoc comments inline, turning your prop definitions into living documentation:

```tsx
interface ButtonProps {
  /** Visual style of the button. Defaults to 'primary'. */
  variant?: 'primary' | 'secondary' | 'ghost';
}
```

## Wrapping Up

Building reusable components is as much a design exercise as it is a coding one — and TypeScript is one of the best design partners you can have. It turns implicit assumptions about props and data shapes into explicit, enforced contracts, which means fewer bugs, better autocomplete, and components that are genuinely a joy to consume.

Start small: take one component in your codebase, give it proper types, and notice the difference the next time you reach for it in your editor. Once you experience the confidence that comes from a well-typed component library, there's no going back to loose props and `any`.

The investment in types pays for itself the very first time autocomplete saves you from a typo — and it just keeps paying dividends from there.
