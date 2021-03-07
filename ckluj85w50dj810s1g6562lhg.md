## How To Use Styled-Components In React

Styled components are a CSS-in-JS tool that bridges the gap between components and styling, offering numerous features to get you up and running in styling components in a functional and reusable way. In this article, you’ll learn the basics of styled components and how to properly apply them to your React applications. You should have worked on React previously before going through this tutorial.

At the core of CSS is the capability to target any HTML element — globally — no matter its position in the DOM tree. This can be a hindrance when used with components, because components demand, to a reasonable extent, colocation (i.e. keeping assets such as states and styling) closer to where they’re used (known as localisation).

In React’s own words, styled components are “visual primitives for components”, and their goal is to give us a flexible way to style components. The result is a tight coupling between components and their styles.

## Why Styled Components?
Apart from helping you to scope styles, styled components include the following features:

- **Automatic vendor prefixing:**
You can use standard CSS properties, and styled components will add vendor prefixes should they be needed.
- **Unique class names:**
Styled components are independent of each other, and you do not have to worry about their names because the library handles that for you.
- **Elimination of dead styles:**
Styled components remove unused styles, even if they’re declared in your code.

## Installation
Installing styled components is easy. You can do it through a CDN or with a package manager such as Yarn…
```yarn add styled-components```

Our demo uses  [create-react-app](https://create-react-app.dev) .

## Starting Out
Perhaps the first thing you’ll notice about styled components is their syntax, which can be daunting if you don’t understand the magic behind styled components. To put it briefly, styled components use JavaScript’s  [template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)  to bridge the gap between components and styles. So, when you create a styled component, what you’re actually creating is a React component with styles. It looks like this:
```
import styled from "styled-components";

// Styled component named StyledButton
const StyledButton = styled.button`
  background-color: black;
  font-size: 32px;
  color: white;
`;

function Component() {
  // Use it like any other component.
  return <StyledButton> Login </StyledButton>;
}
```

Here, `StyledButton` is the styled component, and it will be rendered as an HTML button with the contained styles. styled is an internal utility method that transforms the styling from JavaScript into actual CSS.

In raw HTML and CSS, we would have this:
```
button {
  background-color: black;
  font-size: 32px;
  color: white;
}

<button> Login </button>
```

## Adapting Based On Props
Styled components are functional, so we can easily style elements dynamically. Let’s assume we have two types of buttons on our page, one with a black background, and the other blue. We do not have to create two styled components for them; we can adapt their styling based on their props.
```
import styled from "styled-components";

const StyledButton = styled.button`
  min-width: 200px;
  border: none;
  font-size: 18px;
  padding: 7px 10px;
  /* The resulting background color will be based on the bg props. */
  background-color: ${props => props.bg === "black" ? "black" : "blue";
`;

function Profile() {
  return (
    <div>
      <StyledButton bg="black">Button A</StyledButton>
      <StyledButton bg="blue">Button B</StyledButton>
    </div>
  )
}
```

Because `StyledButton` is a React component that accepts props, we can assign a different background color based on the existence or value of the `bg` prop.

You’ll notice, though, that we haven’t given our button a `type`. Let’s do that:
```
function Profile() {
  return (
    <>
      <StyledButton bg="black" type="button">
        Button A
      </StyledButton>
      <StyledButton bg="blue" type="submit" onClick={() => alert("clicked")}>
        Button B
      </StyledButton>
    </>
  );
}
```

Styled components can differentiate between the  [types of props](https://styled-components.com/docs/basics#passed-props)  they receive. They know that `type` is an HTML attribute, so they actually render `<button type="button">Button A</button>`, while using the `bg` prop in their own processing. Notice how we attached an event handler, too?

Speaking of attributes, an extended syntax lets us manage props using the  [attrs constructor](https://styled-components.com/docs/api#attrs) . Check this out:
```
const StyledContainer = styled.section.attrs((props) => ({
  width: props.width || "100%",
  hasPadding: props.hasPadding || false,
}))`
  --container-padding: 20px;
  width: ${(props) => props.width}; // Falls back to 100%
  padding: ${(props) =>
    (props.hasPadding && "var(--container-padding)") || "none"};
`;
```

Notice how we don’t need a ternary when setting the width? That’s because we’ve already set a default for it with `width: props.width || "100%",`. Also, we used  [CSS custom properties](https://styled-components.com/docs/basics#passed-props)  because we can!