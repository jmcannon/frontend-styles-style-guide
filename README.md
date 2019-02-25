[Note: this is a work in progress. Please don't edit this document directly. Feedback and discussion welcome. These are obviously my own opinions; don't mind the language that presumes we've collectively accepted this - I just wrote it in the language of a future style guide.]

<br /><br />

## Background
There is currently no consensus on best practices for styling in the React community. This makes it especially important to define a consistent and sound approach for ourselves.

While debating and revisiting our own approach is always welcome and valuable, it's likely that simply defining an approach and sticking to it will win out over refactors based on style preference or new trending frameworks. More important than how we do it, is that we all do it in the same way. Having said that, we should periodically take the pulse of the React community to check for any emerging consensus on best practices.

<br /><br />

## Write Styles in Javascript
When working with React, CSS can broadly be written in two ways. The first, in its traditional syntax:
```css
.my-class { font-size: 14px, display: flex }
```
And the second, in Javascript object syntax:
```jsx
<MyComponent styles={{ fontSize: 14, display: 'flex' }} />
```

If there is any consensus about React styling, it's that the future of CSS is in Javascript. Traditionally, combining CSS with Javascript or HTML was seen as a violation of separation of concerns. However, React's paradigm already tightly couples JS and markup; it's therefore natural that we should also organize our styling so it lives with the component it styles.

In practice, this is a dramatic win over the mess that maintaining stylesheets usually leads to. It largely solves the problem of a polluted and opaque global namespace by isolating styles inside component files or theme objects. It also simplifies pre-processing by moving nice-to-have functionality from (for example) LESS webpack configuration to javascript libraries.

By making the decision to always write CSS in Javascript, we also promote consistency across styling code. Writing CSS in JS is native to React and results in much cleaner code. It's better to have CSS in JS everywhere, instead of dividing ourselves between the two syntaxes.

We therefore consider writing CSS in its traditional syntax an anti-pattern. There's no reason in our current stack that you should need to.

<br /><br />

## Styled-Components: Write styles as components
[Styled-Components](https://www.styled-components.com/) is a well-documented framework for binding styles to components that enjoys a large and active community. It also gives us access to all the powerful features of LESS or SASS (including auto-prefixing) in a way that feels native to React. Using styled-components, we don't need to define style objects and bind them via classes like this:
```jsx
// AVOID

const styles = {
  myComponent: {
    fontSize: 14,
    display: 'flex',
  }
}

function MyComponent() {
  return <div className={styles.myComponent} />
}
```
With Styled-Components, we can incorporate styles as native components.
```jsx
// GOOD

import styled from 'styled-components';

const MyComponent = styled.div({
  fontSize: 14,
  display: 'flex',
});
```
In the majority of cases, connecting styles to components via classes is just boilerplate. Compared to class-binding, this is a cleaner, more feature-rich approach.

**We are using styled-component's object syntax.** In order to cater to developers who have spent their entire careers writing CSS, much of styled-component's documentation shows the CSS syntax. As explained above, we are opting for a consistent CSS in JS approach.

Styled-Component's popularity has resulted in the wide-spread adoption of its basic API by other frameworks (including Material UI), reducing our dependency on this particular framework.

<br /><br />

## Put conditional styling inside the style object
Sometimes, you need to change the appearance of a component based on its props. You should try to define this behavior inside the style object, which can take a function that passes in props.
```jsx
const MyComponent = styled.div(props => {
  fontSize: props.large ? 21 : 14,
});
```
You should put any attributes that rely on props at the bottom of the style object, separated by a newline to enhance readability. In addition, you should add a Prop Types definition. Currently, our linter won't complain if you don't, but it allows other developers to more easily reuse your component.
```jsx
// GOOD

import PropTypes from 'prop-types';

const MyComponent = styled.div(props => {
  fontFamily: 'helvetica',
  textDecoration: 'underline',

  fontSize: props.large ? 21: 14,
  color: props.color,
  fontWeight: props.heavy ? 'bold' : 'none',
})

MyComponent.propTypes = {
  large: PropTypes.bool,
  color: PropTypes.string,
  heavy: PropTypes.bool,
}
```
For more complex conditional rendering, you may want to write a functional component instead. This syntax is also acceptable inside a styled component:
```jsx
// GOOD

const MyComponent = styled.div(props => {
  backgroundColor:
    (props.type === 'primary' && 'blue')
    || (props.type === 'danger' && 'red')
    || (props.type === 'warning' && 'yellow')
    || 'none'
})
```
If you find your conditional styling logic getting unwieldy, it's likely a clue that you should break things up into smaller components.

<br /><br />

## Separate layout into a container component
CSS styling serves several presentational roles. In particular, styling rules are used to both provide layout (by determining where and how components should render on a page) and appearance (by describing colors, font styles, background images, etc.). The line between these two is often blurry, but it's worthwhile to try and design with the distinction in mind.

When styling a component, imagine it were plucked from your current design and placed somewhere else. What styles are intrinsic to how that component should look in _any_ context? Alternatively, what styles are simply positioning that component on the current page?

The following example mixes styling and layout.
```jsx
// AVOID

<MyComponent />

const MyComponent = styled.div({
  color: 'red',
  border: '1px solid #ddd',
  borderRadius: 10,
  padding: 20,
  marginTop: 50,
  marginLeft: 'auto',
  marginRight: 'auto,
});
```
Notice how we couldn't just put this component somewhere else. Its margin definitions would need to be rewritten to correctly render in a different context.

A better approach is to extract the layout styling into a container component. Using containers for layout is also the paradigm around which flexbox is designed, so you probably shouldn't be relying on `margin: auto` or `float` - those result in components that define their own layout and therefore can't easily be reused. It also makes adding surrounding components more difficult.
```jsx
// GOOD

<CenteredContent>
  <MyComponent />
</CenteredContent>

const CenteredContent = styled.div({
  display: 'flex',
  marginTop: 50,
  justifyContent: 'center',
});

const MyComponent = styled.div({
  color: 'red',
  border: '1px solid #ddd',
  borderRadius: 10,
  padding: 20,
});
```

<br /><br />

## Use descriptive names for layout components
The Styled-Components library eliminates a lot of boilerplate code connecting classes with components. However, this comes at the cost of having to come up with names for lots of visual components.

For components that handle layout, use names that describe the layout. `CenteredContent`, `RightAlignedText`, `ScrollableContainer`, and `ThreeWideGrid` are all good names. It's okay to be a little verbose: the goal is to convey what the layout component is doing without forcing another developer to comb through style definitions.

For components that are providing styling (as opposed to layout), it's okay to use a semantic name (as you probably would if you were writing a functional component). `UserName` or `BoldBlueText` could both be acceptable depending on the context.

This is an example of poorly named markup:

```jsx
// AVOID

<Students>
  <StudentContainer>

    <StudentInfo>
      <Name>Justin</Name>
      <Status>online</Status>
    </StudentInfo>

    <Actions>
      <AddIcon />
      <DeleteIcon />
    </Actions>

    <EditIcon />

  </StudentContainer>
</Students>
```
It's hard to understand the markup above without the style definitions. This is much better:
```jsx
// GOOD

<Grid>
  <GridItem>

    <LeftAlignedContent>
      <Name>Justin</Name>
      <Status>online</Status>
    </LeftAlignedContent>

    <RightAlignedContent>
      <AddIcon />
      <DeleteIcon />
    </RightAlignedContent>

    <FloatingCenteredButton>
      <EditIcon />
    </FloatingCenteredButton>

  </GridItem>
</Grid>
```
If I were responsible for modifying another developer's code, I could quickly orient myself with the second example. You will inevitably need to inspect style definitions, but having markup that explains itself helps us work faster.

<br /><br />

## Using inline styles as a last resort
Despite a decade of prevailing frontend wisdom, inline styles are not inherently evil. After all, if we're now colocating markup, js, and css, it makes sense that we describe a component's style directly in its markup definition.

In practice, inline styles can lead to messy and unreadable render functions. If we are naming our styled components well, then the sacrifice to locality that we make by pulling styles out of markup is minimal. Inline styles also tend to result in components that become dominated by confusing conditional rendering code that attempt to create and modify style objects on the fly. Stateful component code should be primarily concerned with business logic.

However, we're not classifying inline styling as an anti-pattern. In practice, having a light-weight way to apply one-off styling is extremely valuable. As a rule of thumb, you probably want to limit your inline styles to no more than three attributes.

Inline styles are useful for one ubiquitous case in particular. Often, after correctly styling our component by leaving out layout attributes, we need to adjust its margin relative to its surrounding. This is layout, but creating a container component just to adjust a component's margin is awkward and overkill. An inlined margin is a good solution in this case. (If you're confident that this component is never going to be reused elsewhere, then including the margin in the component's styles is preferred.)

```jsx
// OKAY

<MyComponent style={{ marginLeft: 10 }} />
```

Resist the temptation to use inline styles to create many different versions of a component. It's better to add to your component's props API. So, instead of this:
```jsx
// AVOID

<MyComponent />
<MyComponent style={{ color: 'red', fontWeight: 'bold }} />
```
Do this:
```jsx
// GOOD

<MyComponent />
<MyComponent error />

const MyComponent = styled.div(props => {
  fontSize: 14,
  padding: 10,

  color: props.error ? 'red' : 'black',
  fontWeight: props.error ? 'bold' : 'normal',
});
```

<br /><br />

## Use pseudo-classes and selectors, but don't perform magic
You can use pseudo-elements like `:hover` in Styled-Components:
```jsx
const MyComponent = styled.div({
  backgroundColor: 'blue',

  ':hover': {
    backgroundColor: 'red',
  },
})
```
This is great for simple, common functionality that we'd otherwise have to write logic for. It's worth learning
what [pseudo-classes](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Pseudo-classes_and_pseudo-elements) are available to you. Savvy use of CSS can save you a lot of work writing UI code.

However, you can be too savvy. The internet is full of mind-blowing styling hacks that do all kinds of things you wouldn't think were possible with CSS alone. And that's the problem. It won't be obvious to other developers that certain behaviors are coming from styling definitions. If you decide that some neat CSS trick is worth the obscurity, it's worth leaving a comment in the component file pointing this out.

Styled-Components offers full access to LESS and SASS's augmented stylesheet features. This means you can perform nested styling:

```jsx
// GENERALLY AVOID

const MyComponent = styled.div({
  backgroundColor: 'blue',

  '>span': {
    color: 'red',
  },

  'div h2>span': {
    color: 'green',
  },
})
```
You should generally avoid this feature. It's usually better to create new styled components with useful names. However, if there's a case in which you have some very tightly-coupled markup that's not going to be reused, this can be a good option. For example, I used this pattern to style a legal document that contained headers, paragraphs, and lists. This was a one-off styling requirement and this turned out to be an elegant solution.
```jsx
const LegalDocument = styled.div({
   fontSize: 14,

   'h1, p, ul': {
     marginBottom: 20,
   },

   h1: {
     fontSize: 21,
     fontWeight: 'bold',
   },

   ul: {
     marginLeft: 30,
   }
})
```

<br /><br />

## Use flexbox for layout
In general, you shouldn't be relying on things like `textAlign`, `margin: auto`, or `float` to position things. These are attributes that let components layout themselves, making it hard to reuse them.  [Learning flex well](https://yoksel.github.io/flex-cheatsheet/) will **greatly** increase your speed as a frontend developer.

<br /><br />

## Responsive design over breakpoints. Breakpoints over mobile detection.
You should strive to build flex layouts that automatically collapse as screen sizes shrink. You shouldn't think of the app rendering at discrete widths, but instead try to build a UI that elegantly responds to the window at any width. (A good exercise is to resize a browser window with your mouse.)

Dynamic designs will inevitably require hard-coded breakpoints. That's okay. Define the breakpoints according to window width. It's an anti-pattern to explicitly check for mobile browsers for styling purposes.

<br /><br />

## Put styles at the bottom
Styled components can live in their own files and be imported like any other component - but since they're often small and only serve to style a larger component, it makes sense to include multiple styled components in the same file as the parent component. Put these at the bottom - they should be secondary to the main markup and logic, and developers will know where to find them should they need to edit or inspect styles.

