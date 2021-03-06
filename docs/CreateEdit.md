---
layout: default
title: "The Create and Edit Views"
---

# The Create and Edit Views

The Create and Edit views both display a form, initialized with an empty record (for the Create view) or with a record fetched from the API (for the Edit view). The `<Create>` and `<Edit>` components then delegate the actual rendering of the form to a form component - usually `<SimpleForm>`. This form component uses its children ([`<Input>`](./Inputs.md) components) to render each form input.

![post creation form](./img/create-view.png)

![post edition form](./img/edit-view.png)

## The `<Create>` and `<Edit>` components

The `<Create>` and `<Edit>` components render the page title and actions, and fetch the record from the data provider. They are not responsible for rendering the actual form - that's the job of their child component (usually `<SimpleForm>`), to which they pass the `record` as prop.

Here are all the props accepted by the `<Create>` and `<Edit>` components:

* [`title`](#page-title)
* [`actions`](#actions)
* [`aside`](#aside-component)
* [`successMessage`](#success-message)
* [`component`](#component)
* [`undoable`](#undoable) (`<Edit>` only)

Here is the minimal code necessary to display a form to create and edit comments:

{% raw %}
```jsx
// in src/App.js
import React from 'react';
import { Admin, Resource } from 'react-admin';
import jsonServerProvider from 'ra-data-json-server';

import { PostCreate, PostEdit } from './posts';

const App = () => (
    <Admin dataProvider={jsonServerProvider('https://jsonplaceholder.typicode.com')}>
        <Resource name="posts" create={PostCreate} edit={PostEdit} />
    </Admin>
);

export default App;

// in src/posts.js
import React from 'react';
import { Create, Edit, SimpleForm, TextInput, DateInput, ReferenceManyField, Datagrid, TextField, DateField, EditButton } from 'react-admin';
import RichTextInput from 'ra-input-rich-text';

export const PostCreate = (props) => (
    <Create {...props}>
        <SimpleForm>
            <TextInput source="title" />
            <TextInput source="teaser" options={{ multiLine: true }} />
            <RichTextInput source="body" />
            <DateInput label="Publication date" source="published_at" defaultValue={new Date()} />
        </SimpleForm>
    </Create>
);

export const PostEdit = (props) => (
    <Edit {...props}>
        <SimpleForm>
            <TextInput disabled label="Id" source="id" />
            <TextInput source="title" validate={required()} />
            <TextInput multiline source="teaser" validate={required()} />
            <RichTextInput source="body" validate={required()} />
            <DateInput label="Publication date" source="published_at" />
            <ReferenceManyField label="Comments" reference="comments" target="post_id">
                <Datagrid>
                    <TextField source="body" />
                    <DateField source="created_at" />
                    <EditButton />
                </Datagrid>
            </ReferenceManyField>
        </SimpleForm>
    </Edit>
);
```
{% endraw %}

That's enough to display the post edit form:

![post edition form](./img/post-edition.png)

**Tip**: You might find it cumbersome to repeat the same input components for both the `<Create>` and the `<Edit>` view. In practice, these two views almost never have exactly the same form inputs. For instance, in the previous snippet, the `<Edit>` views shows related comments to the current post, which makes no sense for a new post. Having two separate sets of input components for the two views is therefore a deliberate choice. However, if you have the same set of input components, export them as a custom Form component to avoid repetition.

 `<Create>` accepts a `record` prop, to initialize the form based on an value object.

### Page Title

By default, the title for the Create view is "Create [resource_name]", and the title for the Edit view is "Edit [resource_name] #[record_id]".

You can customize this title by specifying a custom `title` prop:

```jsx
export const PostEdit = (props) => (
    <Edit title="Post edition" {...props}>
        ...
    </Edit>
);
```

More interestingly, you can pass an element as `title`. React-admin clones this element and, in the `<EditView>`, injects the current `record`. This allows to customize the title according to the current record:

```jsx
const PostTitle = ({ record }) => {
    return <span>Post {record ? `"${record.title}"` : ''}</span>;
};
export const PostEdit = (props) => (
    <Edit title={<PostTitle />} {...props}>
        ...
    </Edit>
);
```

### Actions

You can replace the list of default actions by your own element using the `actions` prop:

```jsx
import React from 'react';
import Button from '@material-ui/core/Button';
import { TopToolbar, ShowButton } from 'react-admin';

const PostEditActions = ({ basePath, data, resource }) => (
    <TopToolbar>
        <ShowButton basePath={basePath} record={data} />
        {/* Add your custom actions */}
        <Button color="primary" onClick={customAction}>Custom Action</Button>
    </TopToolbar>
);

export const PostEdit = (props) => (
    <Edit actions={<PostEditActions />} {...props}>
        ...
    </Edit>
);
```

### Aside component

You may want to display additional information on the side of the form. Use the `aside` prop for that, passing the component of your choice:

{% raw %}
```jsx
const Aside = () => (
    <div style={{ width: 200, margin: '1em' }}>
        <Typography variant="h6">Post details</Typography>
        <Typography variant="body2">
            Posts will only be published one an editor approves them
        </Typography>
    </div>
);

const PostEdit = props => (
    <Edit aside={<Aside />} {...props}>
        ...
    </Edit>
```
{% endraw %}

The `aside` component receives the same props as the `Edit` or `Create` child component: `basePath`, `record`, `resource`, and `version`. That means you can display non-editable details of the current record in the aside component:

{% raw %}
```jsx
const Aside = ({ record }) => (
    <div style={{ width: 200, margin: '1em' }}>
        <Typography variant="h6">Post details</Typography>
        {record && (
            <Typography variant="body2">
                Creation date: {record.createdAt}
            </Typography>
        )}
    </div>
);
```
{% endraw %}

**Tip**: Always test that the `record` is defined before using it, as react-admin starts rendering the UI before the API call is over.

### Success message

Once the `dataProvider` returns successfully after save, users see a generic notification ("Element created" / "Element updated"). You can customize this message by passing a `successMessage` prop:

```jsx
const PostEdit = props => (
    <Edit successMessage="messages.post_saved" {...props}>
        ...
    </Edit>
```

**Tip**: The message will be translated.

### Component

By default, the Create and Edit views render the main form inside a material-ui `<Card>` element. The actual layout of the form depends on the `Form` component you're using (`<SimpleForm>`, `<TabbedForm>`, or a custom form component).

Some form layouts also use `Card`, in which case the user ends up seeing a card inside a card, which is bad UI. To avoid that, you can override the main form container by passing a `component` prop :

```jsx
// use a div as root component
const PostEdit = props => (
    <Edit component="div" {...props}>
        ...
    </Edit>
);

// use a custom component as root component 
const PostEdit = props => (
    <Edit component={MyComponent} {...props}>
        ...
    </Edit>
);
```

The default value for the `component` prop is `Card`.

### Undoable

By default, the Save and Delete actions are undoable, i.e. react-admin only sends the related request to the data provider after a short delay, during which the user can cancel the action. This is part of the "optimistic rendering" strategy of react-admin ; it makes the user interactions more reactive.

You can disable this behavior by setting `undoable={false}`. With that setting, clicking on the Delete button displays a confirmation dialog. Both the Save and the Delete actions become blocking, and delay the refresh of the screen until the data provider responds.

```jsx
const PostEdit = props => (
    <Edit undoable={false} {...props}>
        ...
    </Edit>
```

**Tip**: If you want a confirmation dialog for the Delete button but don't mind undoable Edits, then pass a [custom toolbar](#toolbar) to the form, as follows:

```jsx
import React from 'react';
import {
    Toolbar,
    SaveButton,
    DeleteButton,
    Edit,
    SimpleForm,
} from 'react-admin';
import { makeStyles } from '@material-ui/core/styles';

const useStyles = makeStyles({
    toolbar: {
        display: 'flex',
        justifyContent: 'space-between',
    },
});

const CustomToolbar = props => (
    <Toolbar {...props} classes={useStyles()}>
        <SaveButton />
        <DeleteButton undoable={false} />
    </Toolbar>
);

const PostEdit = props => (
    <Edit {...props}>
        <SimpleForm toolbar={<CustomToolbar />}>
            ...
        </SimpleForm>
    </Edit>
);
```

## Prefilling a `<Create>` Record

You may need to prepopulate a record based on another one. For that use case, use the `<CloneButton>` component. It expects a `record` and a `basePath` (usually injected to children of `<Datagrid>`, `<SimpleForm>`, `<SimpleShowLayout>`, etc.), so it's as simple to use as a regular field or input.

For instance, to allow cloning all the posts from the list:

```jsx
import React from 'react';
import { List, Datagrid, TextField, CloneButton } from 'react-admin';

const PostList = props => (
    <List {...props}>
        <Datagrid>
            <TextField source="title" />
            <CloneButton />
        </Datagrid>
    </List>
);
```

Alternately, you may need to prepopulate a record based on a *related* record. For instance, in a `PostList` component, you may want to display a button to create a comment related to the current post. Clicking on that button would lead to a `CommentCreate` page where the `post_id` is preset to the id of the Post.

**Note**  `<CloneButton>` is designed to be used in an edit view `<Actions>` component, not inside a `<Toolbar>`. The `Toolbar` is basically for submitting the form, not for going to another resource.

By default, the `<Create>` view starts with an empty `record`. However, if the `location` object (injected by [react-router-dom](https://reacttraining.com/react-router/web/api/location)) contains a `record` in its `state`, the `<Create>` view uses that `record` instead of the empty object. That's how the `<CloneButton>` works behind the hood.

That means that if you want to create a link to a creation form, presetting *some* values, all you have to do is to set the location `state`. `react-router-dom` provides the `<Link>` component for that:

{% raw %}
```jsx
import React from 'react';
import { Datagrid } from 'react-admin';
import Button from '@material-ui/core/Button';
import { Link } from 'react-router-dom';

const CreateRelatedCommentButton = ({ record }) => (
    <Button
        component={Link}
        to={{
            pathname: '/comments/create',
            state: { record: { post_id: record.id } },
        }}
    >
        Write a comment for that post
    </Button>
);

export default PostList = props => (
    <List {...props}>
        <Datagrid>
            ...
            <CreateRelatedCommentButton />
        </Datagrid>
    </List>
)
```
{% endraw %}

**Tip**: To style the button with the main color from the material-ui theme, use the `Link` component from the `react-admin` package rather than the one from `react-router-dom`.

**Tip**: The `<Create>` component also watches the "source" parameter of `location.search` (the query string in the URL) in addition to `location.state` (a cross-page message hidden in the router memory). So the `CreateRelatedCommentButton` could also be written as:

{% raw %}
```jsx
import React from 'react';
import Button from '@material-ui/core/Button';
import { Link } from 'react-router-dom';

const CreateRelatedCommentButton = ({ record }) => (
    <Button
        component={Link}
        to={{
            pathname: '/comments/create',
            search: `?source=${JSON.stringify({ post_id: record.id })}`,
        }}
    >
        Write a comment for that post
    </Button>
);
```
{% endraw %}

## The `<EditGuesser>` component

Instead of a custom `Edit`, you can use the `EditGuesser` to determine which inputs to use based on the data returned by the API.

```jsx
// in src/App.js
import React from 'react';
import { Admin, Resource, EditGuesser } from 'react-admin';
import jsonServerProvider from 'ra-data-json-server';

const App = () => (
    <Admin dataProvider={jsonServerProvider('https://jsonplaceholder.typicode.com')}>
        <Resource name="posts" edit={EditGuesser} />
    </Admin>
);
```

Just like `Edit`, `EditGuesser` fetches the data. It then analyzes the response, and guesses the inputs it should use to display a basic form with the data. It also dumps the components it has guessed in the console, where you can copy it into your own code. Use this feature to quickly bootstrap an `Edit` on top of an existing API, without adding the inputs one by one.

![Guessed Edit](./img/guessed-edit.png)

React-admin provides guessers for the `List` view (`ListGuesser`), the `Edit` view (`EditGuesser`), and the `Show` view (`ShowGuesser`).

**Tip**: Do not use the guessers in production. They are slower than manually-defined components, because they have to infer types based on the content. Besides, the guesses are not always perfect.

## The `<SimpleForm>` component

The `<SimpleForm>` component receives the `record` as prop from its parent component. It is responsible for rendering the actual form. It is also responsible for validating the form data. Finally, it receives a `handleSubmit` function as prop, to be called with the updated record as argument when the user submits the form.

The `<SimpleForm>` renders its child components line by line (within `<div>` components). It accepts Input and Field components as children. It relies on `react-final-form` for form handling.

![post edition form](./img/post-edition.png)

By default the `<SimpleForm>` submits the form when the user presses `ENTER`. If you want
to change this behaviour you can pass `false` for the `submitOnEnter` property, and the user will only be able to submit by pressing the save button. This can be useful e.g. if you have an input widget using `ENTER` for a special function.

Here are all the props you can set on the `<SimpleForm>` component:

* [`initialValues`](#default-values)
* [`validate`](#validation)
* [`submitOnEnter`](#submit-on-enter)
* [`redirect`](#redirection-after-submission)
* [`toolbar`](#toolbar)
* [`variant`](#variant)
* [`margin`](#margin)

```jsx
export const PostCreate = (props) => (
    <Create {...props}>
        <SimpleForm>
            <TextInput source="title" />
            <RichTextInput source="body" />
            <NumberInput source="nb_views" />
        </SimpleForm>
    </Create>
);
```

**Tip**: `Create` and `Edit` inject more props to their child. So `SimpleForm` also expects these props to be set (but you shouldn't set them yourself):

* `save`: The function invoked when the form is submitted.
* `saving`: A boolean indicating whether a save operation is ongoing.

### Label Decoration

`<SimpleForm>` scans its children for the `addLabel` prop, and automatically wraps a child in a `<Labeled>` component when found. This displays a label on top of the child, based on the `label` prop. This is not necessary for `<Input>` components, as they already contain their label. Also, all the react-admin `<Field>` components have a default prop `addLabel: true`, which explains why react-admin shows a label on top of Fields when they are used as children of `<SimpleForm>`. 

For your own components that don't include a label by default, set the `addLabel` prop if you want to use them as `<SimpleForm>` children.

```jsx
const IdentifierField = ({ record }) => (
    <Typography>{record.id}</Typography>
);

const BodyField = ({ record }) => (
    <Identifier label="body">
        <Typography>
            {record.body}
        </Typography>
    </Identifier>
);

const PostEdit = (props) => (
    <Create {...props}>
        <SimpleForm>
            <IdentifierField addLabel label="Identifier"> {/* SimpleForm will add a label */}
            <TextField source="title" /> {/* SimpleForm will add a label, too (TextField has addLabel:true in defaultProps) */}
            <BodyField /> {/* SimpleForm will NOT add a label */}
            <NumberInput source="nb_views" /> {/* SimpleForm will NOT add a label */}
        </SimpleForm>
    </Create>
);
```

## The `<TabbedForm>` component

Just like `<SimpleForm>`, `<TabbedForm>` receives the `record` prop, renders the actual form, and handles form validation on submit. However, the `<TabbedForm>` component renders inputs grouped by tab. The tabs are set by using `<FormTab>` components, which expect a `label` and an `icon` prop.

![tabbed form](./img/tabbed-form.gif)

By default the `<TabbedForm>` submits the form when the user presses `ENTER`, if you want
to change this behaviour you can pass `false` for the `submitOnEnter` property.

Here are all the props accepted by the `<TabbedForm>` component:

* [`initialValues`](#default-values)
* [`validate`](#validation)
* [`submitOnEnter`](#submit-on-enter)
* [`redirect`](#redirection-after-submission)
* [`tabs`](#tabbed-form-tabs)
* [`toolbar`](#toolbar)
* [`variant`](#variant)
* [`margin`](#margin)
* `save`: The function invoked when the form is submitted. This is passed automatically by `react-admin` when the form component is used inside `Create` and `Edit` components.
* `saving`: A boolean indicating whether a save operation is ongoing. This is passed automatically by `react-admin` when the form component is used inside `Create` and `Edit` components.

{% raw %}
```jsx
import React from 'react';
import {
    TabbedForm,
    FormTab,
    Edit,
    Datagrid,
    TextField,
    DateField,
    TextInput,
    ReferenceManyField,
    NumberInput,    
    DateInput,
    BooleanInput,
    EditButton
} from 'react-admin';

export const PostEdit = (props) => (
    <Edit {...props}>
        <TabbedForm>
            <FormTab label="summary">
                <TextInput disabled label="Id" source="id" />
                <TextInput source="title" validate={required()} />
                <TextInput multiline source="teaser" validate={required()} />
            </FormTab>
            <FormTab label="body">
                <RichTextInput source="body" validate={required()} addLabel={false} />
            </FormTab>
            <FormTab label="Miscellaneous">
                <TextInput label="Password (if protected post)" source="password" type="password" />
                <DateInput label="Publication date" source="published_at" />
                <NumberInput source="average_note" validate={[ number(), minValue(0) ]} />
                <BooleanInput label="Allow comments?" source="commentable" defaultValue />
                <TextInput disabled label="Nb views" source="views" />
            </FormTab>
            <FormTab label="comments">
                <ReferenceManyField reference="comments" target="post_id" addLabel={false}>
                    <Datagrid>
                        <TextField source="body" />
                        <DateField source="created_at" />
                        <EditButton />
                    </Datagrid>
                </ReferenceManyField>
            </FormTab>
        </TabbedForm>
    </Edit>
);
```
{% endraw %}

To style the tabs, the `<FormTab>` component accepts two props:

- `className` is passed to the tab *header*
- `contentClassName` is passed to the tab *content*

### Label Decoration

`<FormTab>` scans its children for the `addLabel` prop, and automatically wraps a child in a `<Labeled>` component when found. This displays a label on top of the child, based on the `label` prop. This is not necessary for `<Input>` components, as they already contain their label. Also, all the react-admin `<Field>` components have a default prop `addLabel: true`, which explains why react-admin shows a label on top of Fields when they are used as children of `<FormTab>`. 

For your own components that don't include a label by default, set the `addLabel` prop if you want to use them as `<FormTab>` children.

```jsx
const IdentifierField = ({ record }) => (
    <Typography>{record.id}</Typography>
);

const BodyField = ({ record }) => (
    <Identifier label="body">
        <Typography>
            {record.body}
        </Typography>
    </Identifier>
);

const PostEdit = (props) => (
    <Create {...props}>
        <TabbeForm>
            <FormTab label="main">
                <IdentifierField addLabel label="Identifier"> {/* FormTab will add a label */}
                <TextField source="title" /> {/* FormTab will add a label, too (TextField has addLabel:true) in defaultProps */}
                <BodyField /> {/* FormTab will NOT add a label */}
                <NumberInput source="nb_views" /> {/* FormTab will NOT add a label */}
            </FormTab>
        </TabbeForm>
    </Create>
);
```

### TabbedFormTabs

By default `<TabbedForm>` uses `<TabbedFormTabs>`, an internal react-admin component, to renders tabs. You can pass a custom component as the `tabs` prop to override the default component. Besides, props from `<TabbedFormTabs>` are passed to material-ui's `<Tabs>` component inside `<TabbedFormTabs>`.

The following example shows how to make use of scrollable `<Tabs>`. Pass the `scrollable` prop to `<TabbedFormTabs>` and pass that as the `tabs` prop to `<TabbedForm>`

```jsx
import React from 'react';
import {
    Edit,
    TabbedForm,
    TabbedFormTabs,
} from 'react-admin';

export const PostEdit = (props) => (
    <Edit {...props}>
        <TabbedForm tabs={<TabbedFormTabs scrollable={true} />}>
            ...
        </TabbedForm>
    </Edit>
);
```

## Default Values

To define default values, you can add a `initialValues` prop to form components (`<SimpleForm>`, `<Tabbedform>`, etc.), or add a `defaultValue` to individual input components. Let's see each of these options.

**Note**: on RA v2 the `initialValues` used to be named `defaultValue`

### Global Default Value

The value of the form `initialValues` prop is an object, or a function returning an object, specifying default values for the created record. For instance:

```jsx
const postDefaultValue = { created_at: new Date(), nb_views: 0 };
export const PostCreate = (props) => (
    <Create {...props}>
        <SimpleForm initialValues={postDefaultValue}>
            <TextInput source="title" />
            <RichTextInput source="body" />
            <NumberInput source="nb_views" />
        </SimpleForm>
    </Create>
);
```

**Tip**: You can include properties in the form `initialValues` that are not listed as input components, like the `created_at` property in the previous example.

### Per Input Default Value

Alternatively, you can specify a `defaultValue` prop directly in `<Input>` components. Default value can be a scalar, or a function returning a scalar.  React-admin will merge the input default values with the form default value (input > form):

```jsx
export const PostCreate = (props) => (
    <Create {...props}>
        <SimpleForm>
            <TextInput disabled source="id" defaultValue={() => uuid()}/>
            <TextInput source="title" />
            <RichTextInput source="body" />
            <NumberInput source="nb_views" defaultValue={0} />
        </SimpleForm>
    </Create>
);
```

## Validation

React-admin relies on [react-final-form](https://github.com/final-form/react-final-form) for the validation.

To validate values submitted by a form, you can add a `validate` prop to the form component, to individual inputs, or even mix both approaches.

### Global Validation

The value of the form `validate` prop must be a function taking the record as input, and returning an object with error messages indexed by field. For instance:

```jsx
const validateUserCreation = (values) => {
    const errors = {};
    if (!values.firstName) {
        errors.firstName = ['The firstName is required'];
    }
    if (!values.age) {
        errors.age = ['The age is required'];
    } else if (values.age < 18) {
        errors.age = ['Must be over 18'];
    }
    return errors
};

export const UserCreate = (props) => (
    <Create {...props}>
        <SimpleForm validate={validateUserCreation}>
            <TextInput label="First Name" source="firstName" />
            <TextInput label="Age" source="age" />
        </SimpleForm>
    </Create>
);
```

**Tip**: The props you pass to `<SimpleForm>` and `<TabbedForm>` are passed to the `<Form>` of `react-final-form`.

### Per Input Validation: Built-in Field Validators

Alternatively, you can specify a `validate` prop directly in `<Input>` components, taking either a function, or an array of functions. React-admin already bundles a few validator functions, that you can just require, and use as input-level validators:

* `required(message)` if the field is mandatory,
* `minValue(min, message)` to specify a minimum value for integers,
* `maxValue(max, message)` to specify a maximum value for integers,
* `minLength(min, message)` to specify a minimum length for strings,
* `maxLength(max, message)` to specify a maximum length for strings,
* `number(message)` to check that the input is a valid number,
* `email(message)` to check that the input is a valid email address,
* `regex(pattern, message)` to validate that the input matches a regex,
* `choices(list, message)` to validate that the input is within a given list,

Example usage:

```jsx
import {
    required,
    minLength,
    maxLength,
    minValue,
    maxValue,
    number,
    regex,
    email,
    choices
} from 'react-admin';

const validateFirstName = [required(), minLength(2), maxLength(15)];
const validateEmail = email();
const validateAge = [number(), minValue(18)];
const validateZipCode = regex(/^\d{5}$/, 'Must be a valid Zip Code');
const validateSex = choices(['m', 'f'], 'Must be Male or Female');

export const UserCreate = (props) => (
    <Create {...props}>
        <SimpleForm>
            <TextInput label="First Name" source="firstName" validate={validateFirstName} />
            <TextInput label="Email" source="email" validate={validateEmail} />
            <TextInput label="Age" source="age" validate={validateAge}/>
            <TextInput label="Zip Code" source="zip" validate={validateZipCode}/>
            <SelectInput label="Sex" source="sex" choices={[
                { id: 'm', name: 'Male' },
                { id: 'f', name: 'Female' },
            ]} validate={validateSex}/>
        </SimpleForm>
    </Create>
);
```

**Tip**: If you pass a function as a message, react-admin calls this function with `{ args, value, values,translate, ...props }` as argument. For instance:

```jsx
const message = ({ translate }) => translate('myroot.validation.email_invalid');
const validateEmail = email(message);
```

### Per Input Validation: Custom Function Validator

You can also define your own validator functions. These functions should return `undefined` when there is no error, or an error string.


```jsx
const required = (message = 'Required') =>
    value => value ? undefined : message;
const maxLength = (max, message = 'Too short') =>
    value => value && value.length > max ? message : undefined;
const number = (message = 'Must be a number') =>
    value => value && isNaN(Number(value)) ? message : undefined;
const minValue = (min, message = 'Too small') =>
    value => value && value < min ? message : undefined;

const ageValidation = (value, allValues) => {
    if (!value) {
        return 'The age is required';
    }
    if (value < 18) {
        return 'Must be over 18';
    }
    return [];
};

const validateFirstName = [required(), maxLength(15)];
const validateAge = [required(), number(), ageValidation];

export const UserCreate = (props) => (
    <Create {...props}>
        <SimpleForm>
            <TextInput label="First Name" source="firstName" validate={validateFirstName} />
            <TextInput label="Age" source="age" validate={validateAge}/>
        </SimpleForm>
    </Create>
);
```

React-admin will combine all the input-level functions into a single function looking just like the previous one.

Input validation functions receive the current field value, and the values of all fields of the current record. This allows for complex validation scenarios (e.g. validate that two passwords are the same).

**Tip**: If your admin has multi-language support, validator functions should return message *identifiers* rather than messages themselves. React-admin automatically passes these identifiers to the translation function: 

```jsx
// in validators/required.js
const required = () => (value, allValues, props) =>
    value
        ? undefined
        : 'myroot.validation.required';

// in i18n/en.json
export default {
    myroot: {
        validation: {
            required: 'Required field',
        }
    }
}
```

If the translation depends on a variable, the validator can return an object rather than a translation identifier:

```jsx
// in validators/minLength.js
const minLength = (min) => (value, allValues, props) => 
    value.length >= min
        ? undefined
        : { message: 'myroot.validation.minLength', args: { min } };

// in i18n/en.js
export default {
    myroot: {
        validation: {
            minLength: 'Must be %{min} characters at least',
        }
    }
}
```

See the [Translation documentation](Translation.md#translation-messages) for details.

**Tip**: Make sure to define validation functions or array of functions in a variable, instead of defining them directly in JSX. This can result in a new function or array at every render, and trigger infinite rerender.

{% raw %}
```jsx
const validateStock = [required(), number(), minValue(0)];

export const ProductEdit = ({ ...props }) => (
    <Edit {...props}>
        <SimpleForm initialValues={{ stock: 0 }}>
            ...
            {/* do this */}
            <NumberInput source="stock" validate={validateStock} />
            {/* don't do that */}
            <NumberInput source="stock" validate={[required(), number(), minValue(0)]} />
            ...
        </SimpleForm>
    </Edit>
);
```
{% endraw %}

**Tip**: The props of your Input components are passed to a `react-final-form` `<Field>` component.

**Tip**: You can use *both* Form validation and input validation.

## Submit On Enter

By default, pressing `ENTER` in any of the form fields submits the form - this is the expected behavior in most cases. However, some of your custom input components (e.g. Google Maps widget) may have special handlers for the `ENTER` key. In that case, to disable the automated form submission on enter, set the `submitOnEnter` prop of the form component to `false`:

```jsx
export const PostEdit = (props) => (
    <Edit {...props}>
        <SimpleForm submitOnEnter={false}>
            ...
        </SimpleForm>
    </Edit>
);
```

## Redirection After Submission

By default:

- Submitting the form in the `<Create>` view redirects to the `<Edit>` view
- Submitting the form in the `<Edit>` view redirects to the `<List>` view

You can customize the redirection by setting the `redirect` prop of the form component. Possible values are "edit", "show", "list", and `false` to disable redirection. You may also specify a custom path such as `/my-custom-route`. For instance, to redirect to the `<Show>` view after edition:

```jsx
export const PostEdit = (props) => (
    <Edit {...props}>
        <SimpleForm redirect="show">
            ...
        </SimpleForm>
    </Edit>
);
```

You can also pass a custom route (e.g. "/home") or a function as `redirect` prop value. For example, if you want to redirect to a page related to the current object:

```jsx
// redirect to the related Author show page
const redirect = (basePath, id, data) => `/author/${data.author_id}/show`;

export const PostEdit = (props) => {
    <Edit {...props}>
        <SimpleForm redirect={redirect}>
            ...
        </SimpleForm>
    </Edit>
);
```

This affects both the submit button, and the form submission when the user presses `ENTER` in one of the form fields.

## Toolbar

At the bottom of the form, the toolbar displays the submit button. You can override this component by setting the `toolbar` prop, to display the buttons of your choice.

The most common use case is to display two submit buttons in the `<Create>` view:

- one that creates and redirects to the `<Show>` view of the new resource, and
- one that redirects to a blank `<Create>` view after creation (allowing bulk creation)

![Form toolbar](./img/form-toolbar.png)

For that use case, use the `<SaveButton>` component with a custom `redirect` prop:

```jsx
import React from 'react';
import { Create, SimpleForm, SaveButton, Toolbar } from 'react-admin';

const PostCreateToolbar = props => (
    <Toolbar {...props} >
        <SaveButton
            label="post.action.save_and_show"
            redirect="show"
            submitOnEnter={true}
        />
        <SaveButton
            label="post.action.save_and_add"
            redirect={false}
            submitOnEnter={false}
            variant="text"
        />
    </Toolbar>
);

export const PostCreate = (props) => (
    <Create {...props}>
        <SimpleForm toolbar={<PostCreateToolbar />} redirect="show">
            ...
        </SimpleForm>
    </Create>
);
```

Another use case is to remove the `<DeleteButton>` from the toolbar in an edit view. In that case, create a custom toolbar containing only the `<SaveButton>` as child;

```jsx
import React from 'react';
import { Edit, SimpleForm, SaveButton, Toolbar } from 'react-admin';

const PostEditToolbar = props => (
    <Toolbar {...props} >
        <SaveButton />
    </Toolbar>
);

export const PostEdit = (props) => (
    <Edit {...props}>
        <SimpleForm toolbar={<PostEditToolbar />}>
            ...
        </SimpleForm>
    </Edit>
);
```

Here are the props received by the `Toolbar` component when passed as the `toolbar` prop of the `SimpleForm` or `TabbedForm` components:

* `handleSubmitWithRedirect`: The function to call in order to submit the form. It accepts a single parameter overriding the form's default redirect.
* `handleSubmit` which is the same prop as in [`react-final-form`](https://final-form.org/docs/react-final-form/types/FormRenderProps#handlesubmit)
* `invalid`: A boolean indicating whether the form is invalid
* `pristine`: A boolean indicating whether the form is pristine (eg: no inputs have been changed yet)
* `redirect`: The default form's redirect
* `saving`: A boolean indicating whether a save operation is ongoing.
* `submitOnEnter`: A boolean indicating whether the form should be submitted when pressing `enter`

**Tip**: Use react-admin's `<Toolbar>` component instead of material-ui's `<Toolbar>` component. The former builds up on the latter, and adds support for an alternative mobile layout (and is therefore responsive).

**Tip**: Don't forget to also set the `redirect` prop of the Form component to handle submission by the `ENTER` key.

**Tip**: To alter the form values before submitting, you should use the `handleSubmit` prop. See [Altering the Form Values before Submitting](#altering-the-form-values-before-submitting) for more information and examples.

**Tip**: If you want to include a `<CreateButton>` in the `<Toolbar>`, the props injected by `<Toolbar>` to its children (`handleSubmit`, `handleSubmitWithRedirect`, `onSave`, `invalid`, `pristine`, `saving`, and `submitOnEnter`) will cause React warnings. You'll need to wrap `<CreateButton>` in another component and ignore the injected props, as follows:

```jsx
const ToolbarCreateButton = ({
  handleSubmit,
  handleSubmitWithRedirect,
  onSave,
  invalid,
  pristine,
  saving,
  submitOnEnter,
  ...rest
}) => <CreateButton {...rest} />;

const PostEditToolbar = props => (
    <Toolbar {...props} >
        <ToolbarCreateButton />
    </Toolbar>
);
```

## Customizing The Form Layout

You can customize each row in a `<SimpleForm>` or in a `<TabbedForm>` by passing props to the Input components:

* `className`
* [`variant`](#variant)
* [`margin`](#margin)
* [`formClassName`](#formclassname)
* [`fullWidth`](#fullwidth)

You can find more about these props in [the Input documentation](./Inputs.md#common-input-props).

You can also [wrap inputs inside containers](#custom-row-container), or [create a custom Form component](#custom-form-component), alternative to `<SimpleForm>` or `<TabbedForm>`. 

### Variant

By default, react-admin input components use the Material Design "filled" variant. If you want to use the "standard" or "outlined" variants, you can either set the `variant` prop on each Input component individually, or set the `variant` prop directly on the Form component. In that case, the Form component will transmit the `variant` to each Input.

```jsx
export const PostEdit = (props) => (
    <Edit {...props}>
        <SimpleForm variant="standard">
            ...
        </SimpleForm>
    </Edit>
);
```

**Tip**: If your form contains not only Inputs but also Fields, the injection of the `variant` property to the form children will cause a React warning. You'll need to wrap every Field component in another component to ignore the injected `variant` prop, as follows:

```diff
+const TextFieldInForm = ({ variant, ...props }) => <TextField {...props} />;
+TextFieldInForm.defaultProps = TextField.defaultProps;

```jsx
export const PostEdit = (props) => (
    <Edit {...props}>
        <SimpleForm variant="standard">
-           <TextField source="title" />
+           <TextFieldInForm source="title" />
        </SimpleForm>
    </Edit>
);
```

### Margin

By default, react-admin input components use the Material Design "dense" margin. If you want to use the "normal" or "none" margins, you can either set the `margin` prop on each Input component individually, or set the `margin` prop directly on the Form component. In that case, the Form component will transmit the `margin` to each Input.

```jsx
export const PostEdit = (props) => (
    <Edit {...props}>
        <SimpleForm margin="normal">
            ...
        </SimpleForm>
    </Edit>
);
```

### `formClassName`

The input components are wrapped inside a `div` to ensure a good looking form by default. You can pass a `formClassName` prop to the input components to customize the style of this `div`. For example, here is how to display two inputs on the same line:

```jsx
import React from 'react';
import {
    Edit,
    SimpleForm,
    TextInput,
} from 'react-admin';
import { makeStyles } from '@material-ui/core/styles';

const useStyles = makeStyles({
    inlineBlock: { display: 'inline-flex', marginRight: '1rem' },
});

export const UserEdit = props => {
    const classes = useStyles();
    return (
        <Edit {...props}>
            <SimpleForm>
                <TextInput source="first_name" formClassName={classes.inlineBlock} />
                <TextInput source="last_name" formClassName={classes.inlineBlock} />
                {/* This input will be display below the two first ones */}
                <TextInput source="email" type="email" />
            </SimpleForm>
        </Edit>
    )
}
```

### `fullWidth`

If you just need a form row to take the entire form width, use the `fullWidth` prop instead:

```jsx
export const UserEdit = props => (
    <Edit {...props}>
        <SimpleForm>
            <TextInput source="first_name" fullWidth />
            <TextInput source="last_name" fullWidth />
            <TextInput source="email" type="email" fullWidth />
        </SimpleForm>
    </Edit>
);
```

### Custom Row Container

You may want to customize the styles of Input components by wrapping them inside a container with a custom style. Unfortunately, this doesn't work:

```jsx
export const PostCreate = props => (
    <Create {...props}>
        <SimpleForm>
            {/* this does not work */}
            <div className="special-input">
                <TextInput source="title" />
            </div>
            <RichTextInput source="body" />
            <NumberInput source="nb_views" />
        </SimpleForm>
    </Create>
);
```

That's because `<SimpleForm>` clones its children and injects props to them (like `record` or `resource`). Input and Field components expect these props, but DOM elements don't. That means that if you wrap an Input or a Field element in a `<div>`, you'll get a React warning about unrecognized DOM attributes, and an error about missing props in the child.

You can try passing `className` to the Input element directly - all form inputs accept a `className` prop.

Alternatively, you can create a custom Input component:

```jsx
const MyTextInput = props => (
    <div className="special-input">
        <TextInput {...props} />
    </div>
)
export const PostCreate = (props) => (
    <Create {...props}>
        <SimpleForm>
            {/* this works */}
            <MyTextInput source="title" />
            <RichTextInput source="body" />
            <NumberInput source="nb_views" />
        </SimpleForm>
    </Create>
);
```

### Custom Form Component

The `<SimpleForm>` and `<TabbedForm>` layouts are quite simple. In order to better use the screen real estate, you may want to arrange inputs differently, e.g. putting them in groups, adding separators, etc. For that purpose, you need to write a custom form layout, and use it instead of `<SimpleForm>`. 

![custom form layout](./img/custom-form-layout.png)
 
Here is an example of such custom form, taken from the Posters Galore demo. It uses [material-ui's `<Box>` component](https://material-ui.com/components/box/), and it's a good starting point for your custom form layouts.

```jsx
import React from 'react';
import {
    FormWithRedirect,
    DateInput,
    SelectArrayInput,
    TextInput,
    SaveButton,
    DeleteButton,
    NullableBooleanInput,
} from 'react-admin';
import { Typography, Box, Toolbar } from '@material-ui/core';

const segments = [
    { id: 'compulsive', name: 'Compulsive' },
    { id: 'collector', name: 'Collector' },
    { id: 'ordered_once', name: 'Ordered Once' },
    { id: 'regular', name: 'Regular' },
    { id: 'returns', name: 'Returns' },
    { id: 'reviewer', name: 'Reviewer' },
];

const VisitorForm = props => (
    <FormWithRedirect
        {...props}
        render={formProps => (
            // here starts the custom form layout
            <form>
                <Box p="1em">
                    <Box display="flex">
                        <Box flex={2} mr="1em">

                            <Typography variant="h6" gutterBottom>Identity</Typography>

                            <Box display="flex">
                                <Box flex={1} mr="0.5em">
                                    <TextInput source="first_name" resource="customers" fullWidth />
                                </Box>
                                <Box flex={1} ml="0.5em">
                                    <TextInput source="last_name" resource="customers" fullWidth />
                                </Box>
                            </Box>
                            <TextInput source="email" resource="customers" type="email" fullWidth />
                            <DateInput source="birthday" resource="customers" />
                            <Box mt="1em" />

                            <Typography variant="h6" gutterBottom>Address</Typography>

                            <TextInput resource="customers" source="address" multiline fullWidth />
                            <Box display="flex">
                                <Box flex={1} mr="0.5em">
                                    <TextInput source="zipcode" resource="customers" fullWidth />
                                </Box>
                                <Box flex={2} ml="0.5em">
                                    <TextInput source="city" resource="customers" fullWidth />
                                </Box>
                            </Box>
                        </Box>

                        <Box flex={1} ml="1em">
                            
                            <Typography variant="h6" gutterBottom>Stats</Typography>

                            <SelectArrayInput source="groups" resource="customers" choices={segments} fullWidth />
                            <NullableBooleanInput source="has_newsletter" resource="customers" />
                        </Box>

                    </Box>
                </Box>
                <Toolbar>
                    <Box display="flex" justifyContent="space-between" width="100%">
                        <SaveButton
                            saving={formProps.saving}
                            handleSubmitWithRedirect={formProps.handleSubmitWithRedirect}
                        />
                        <DeleteButton record={formProps.record} />
                    </Box>
                </Toolbar>
            </form>
        )}
    />
);
```

This custom form layout component uses the `FormWithRedirect` component, which wraps react-final-form's `Form` component to handle redirection logic. It also uses react-admin's `<SaveButton>` and a `<DeleteButton>`.

**Tip**: When `Input` components have a `resource` prop, they use it to determine the input label. `<SimpleForm>` and `<TabbedForm>` inject this `resource` prop to `Input` components automatically. When you use a custom form layout, pass the `resource` prop manually - unless the `Input` has a `label` prop.

To use this form layout, simply pass it as child to an `Edit` component:

```jsx
const VisitorEdit = props => (
    <Edit {...props}>
        <VisitorForm />
    </Edit>
);
```

**Tip**: `FormWithRedirect` contains some logic that you may not want. In fact, nothing forbids you from using [a react-final-form `Form` component](https://final-form.org/docs/react-final-form/api/Form) as root component for a custom form layout. You'll have to set initial values based the injected `record` prop manually, as follows:

{% raw %}
```jsx
import { sanitizeEmptyValues } from 'react-admin';
import { Form } from 'react-final-form';
import arrayMutators from 'final-form-arrays';
import { CardContent, Typography, Box } from '@material-ui/core';

// the parent component (Edit or Create) injects these props to their child
const VisitorForm = ({ basePath, record, save, saving, version }) => {
    const submit = values => {
        // React-final-form removes empty values from the form state.
        // To allow users to *delete* values, this must be taken into account 
        save(sanitizeEmptyValues(record, values));
    };
    return (
        <Form
            initialValues={record}
            onSubmit={submit}
            mutators={{ ...arrayMutators }} // necessary for ArrayInput
            subscription={defaultSubscription} // don't redraw entire form each time one field changes
            key={version} // support for refresh button
            keepDirtyOnReinitialize
            render={formProps => (
                // render your custom form here
            )}
        />
    );
};
const defaultSubscription = {
    submitting: true,
    pristine: true,
    valid: true,
    invalid: true,
};
```
{% endraw %}

## Displaying Fields or Inputs depending on the user permissions

You might want to display some fields, inputs or filters only to users with specific permissions. 

Before rendering the `Create` and `Edit` components, react-admin calls the `authProvider.getPermissions()` method, and passes the result to the component as the `permissions` prop. It's up to your `authProvider` to return whatever you need to check roles and permissions inside your component.

Here is an example inside a `Create` view with a `SimpleForm` and a custom `Toolbar`:

{% raw %}
```jsx
const UserCreateToolbar = ({ permissions, ...props }) =>
    <Toolbar {...props}>
        <SaveButton
            label="user.action.save_and_show"
            redirect="show"
            submitOnEnter={true}
        />
        {permissions === 'admin' &&
            <SaveButton
                label="user.action.save_and_add"
                redirect={false}
                submitOnEnter={false}
                variant="text"
            />}
    </Toolbar>;

export const UserCreate = ({ permissions, ...props }) =>
    <Create {...props}>
        <SimpleForm
            toolbar={<UserCreateToolbar permissions={permissions} />}
            initialValues={{ role: 'user' }}
        >
            <TextInput source="name" validate={[required()]} />
            {permissions === 'admin' &&
                <TextInput source="role" validate={[required()]} />}
        </SimpleForm>
    </Create>;
```
{% endraw %}

**Tip**: Note how the `permissions` prop is passed down to the custom `toolbar` component.

This also works inside an `Edition` view with a `TabbedForm`, and you can hide a `FormTab` completely:

{% raw %}
```jsx
export const UserEdit = ({ permissions, ...props }) =>
    <Edit title={<UserTitle />} {...props}>
        <TabbedForm initialValues={{ role: 'user' }}>
            <FormTab label="user.form.summary">
                {permissions === 'admin' && <TextInput disabled source="id" />}
                <TextInput source="name" validate={required()} />
            </FormTab>
            {permissions === 'admin' &&
                <FormTab label="user.form.security">
                    <TextInput source="role" validate={required()} />
                </FormTab>}
        </TabbedForm>
    </Edit>;
```
{% endraw %}

## Altering the Form Values Before Submitting

Sometimes, you may want to alter the form values before actually sending them to the `dataProvider`. For those cases, you should know that every button inside a form [Toolbar](#toolbar) receive two props:

* `handleSubmit` which calls the default form save method (provided by react-final-form)
* `handleSubmitWithRedirect` which calls the default form save method and allows to specify a custom redirection
​
Decorating `handleSubmitWithRedirect` with your own logic allows you to alter the form values before submitting. For instance, to set the `average_note` field value just before submission:

```jsx
import React, { useCallback } from 'react';
import { useForm } from 'react-final-form';
import {
    SaveButton,
    Toolbar,
    useCreate,
    useRedirect,
    useNotify,
} from 'react-admin';
​
const SaveWithNoteButton = ({ handleSubmitWithRedirect, ...props }) => {
    const [create] = useCreate('posts');
    const redirectTo = useRedirect();
    const notify = useNotify();
    const { basePath, redirect } = props;
​
    const form = useForm();
​
    const handleClick = useCallback(() => {
        // change the average_note field value
        form.change('average_note', 10);
​
        handleSubmitWithRedirect('edit');
    }, [form]);
​
    // override handleSubmitWithRedirect with custom logic
    return <SaveButton {...props} handleSubmitWithRedirect={handleClick} />;
};
```

This button can be used in the `PostCreateToolbar` component:

```jsx
const PostCreateToolbar = props => (
    <Toolbar {...props}>
        <SaveButton
            label="post.action.save_and_show"
            redirect="show"
            submitOnEnter={true}
        />
        <SaveWithNoteButton
            label="post.action.save_with_average_note"
            redirect="show"
            submitOnEnter={false}
            variant="text"
        />
    </Toolbar>
);
```

**Tip**: Which one of `handleSubmit` and `handleSubmitWithRedirect` should you override? If you want to keep the redirection, override `handleSubmitWithRedirect` just like in the previous example. If you want to disable redirection, or handle it yourself, override `handleSubmit`.  

## Using `onSave` To Alter the Form Submission Behavior

The previous technique works well for altering values. But you may want to call a route before submission, or submit the form to different dataProvider methods/resources depending on the form values. And in this case, wrapping `handleSubmitWithRedirect` does not work, because you don't have control on the submission itself.
​
Instead of *decorating* `handleSubmitWithRedirect`, you can *replace* it, and do the API call manually. You don't have to change anything in the form values in that case. So the previous example can be rewritten as:

```jsx
import React, { useCallback } from 'react';
import { useFormState } from 'react-final-form';
import {
    SaveButton,
    Toolbar,
    useCreate,
    useRedirect,
    useNotify,
} from 'react-admin';
​
const SaveWithNoteButton = props => {
    const [create] = useCreate('posts');
    const redirectTo = useRedirect();
    const notify = useNotify();
    const { basePath, redirect } = props;
    // get values from the form
    const formState = useFormState();
​
    const handleClick = useCallback(
        () => {
            // call dataProvider.create() manually
            create(
                {
                    payload: { data: { ...formState.values, average_note: 10 } },
                },
                {
                    onSuccess: ({ data: newRecord }) => {
                        notify('ra.notification.created', 'info', {
                            smart_count: 1,
                        });
                        redirectTo(redirect, basePath, newRecord.id, newRecord);
                    },
                }
            );
        },
        [create, notify, redirectTo, basePath, formState, redirect]
    );
​
    return <SaveButton {...props} handleSubmitWithRedirect={handleClick} />;
};
```

This technique has a huge drawback, which makes it impractical: by skipping the default `handleSubmitWithRedirect`, this button doesn't trigger form validation. And unfortunately, react-final-form doesn't provide a way to trigger form validation manually.
​
That's why react-admin provides a way to override just the data provider call and its side effects. It's called `onSave`, and here is how you would use it in the previous use case:

```jsx
import React, { useCallback } from 'react';
import {
    SaveButton,
    Toolbar,
    useCreate,
    useRedirect,
    useNotify,
} from 'react-admin';
​
const SaveWithNoteButton = props => {
    const [create] = useCreate('posts');
    const redirectTo = useRedirect();
    const notify = useNotify();
    const { basePath } = props;
​
    const handleSave = useCallback(
        (values, redirect) => {
            create(
                {
                    payload: { data: { ...values, average_note: 10 } },
                },
                {
                    onSuccess: ({ data: newRecord }) => {
                        notify('ra.notification.created', 'info', {
                            smart_count: 1,
                        });
                        redirectTo(redirect, basePath, newRecord.id, newRecord);
                    },
                }
            );
        },
        [create, notify, redirectTo, basePath]
    );
​
    // set onSave props instead of handleSubmitWithRedirect
    return <SaveButton {...props} onSave={handleSave} />;
};
```

The `onSave` value should be a function expecting 2 arguments: the form values to save, and the redirection to perform.
