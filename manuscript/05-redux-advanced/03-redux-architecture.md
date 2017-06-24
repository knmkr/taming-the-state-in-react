# Redux State as Architecture

The book taught you the practical usage of Redux. You have learned about the main parts in the Redux state management architecture: actions, reducers and the store.

{title="Code Playground",lang="javascript"}
~~~~~~~~
Action -> Reducer(s) -> Store
~~~~~~~~

The chain is connected to the view by something (e.g. react-redux with `mapStateToProps()` and `mapDispatchToProps()`) that enables you to write connected components. These components have access to the Redux store. They are used to receive state or to alter the state. They are a specialized case of a container component in the presenter and container pattern when using components.

{title="Code Playground",lang="javascript"}
~~~~~~~~
View -> (mapDispatchToProps) -> Action -> Reducer(s) -> Store -> (mapStateToProps) -> View
~~~~~~~~

All other components are not aware of any local or sophisticated state management solution. They only receive props, except they have their own local state management (such as `this.state` and `this.setState()` in React).

State can be received directly by operating on the state object or indirectly by selecting it with selectors.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// directly from state object
state.something;

// indirectly from state object via selector
const getSomething = (state) => state.something;
~~~~~~~~

State can be altered by dispatching an action directly or by using action creators that return an action object.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// dispatching an action directly
dispatch({ type: 'ANY_TYPE', payload: anyPayload });

// dispatching an action indirectly via action creator
function doAnything(payload) {
  return {
    type: 'ANY_TYPE',
    payload,
  };
}
dispatch(doAnything(anyPayload));
~~~~~~~~

In order to keep your state predictable and manageable in the reducers, you can apply techniques for an improved state structure. You can normalize your state to have always a single source of truth. That means you don't have to operate on duplicated entities, but only on one reference of the entity. In addition, it keeps the state flat. It is easier to manage only by using spread operators.

Around these practical usages, you have learned several supporting techniques. There are tons of opinionated ways to organize your folders and files. The book showcased two of the main approaches, but they vary in their execution from developer to developer, team to team or company to company. Nevertheless, you should always bear in mind to keep Redux at a top level. It is not used to manage the state of one single component. Instead it is used to wire dedicated components to the store in order to enable them to alter and to retrieve the state from it.

Coupling actions and reducers is fine, but always think twice when adding another action type. For instance, perhaps a action type could be reused in another reducer. When reusing action types, you avoid to end up with fat thunks when using Redux thunk. Instead of dispatching several actions, your thunk could dispatch only one abstract action that is reused in more than one reducer.

You have learned that you can plan your state management ahead. There are use cases where local state makes more sense than sophisticated state. Both can be used and should be used in a scaling application. By combining local state to the native local storage of the browser, you can give the user of your application an improved UX. In addition, you can plan the state ahead too. Think about view state and entitiy state and where it should live in your application. You can give your reducers differenct domains as their ownership such as `todoReducer`, filterReducer and notificationReducer. However, once you have planned your state management and state, don't stick to it. When slaing your application, always revisit those things to apply refactorings. That will help you to keep your state manageable, maintainable and predictable in the long run.

# Hands On: Hacker News with Redux

In this chapter, you will be guided to build your own Hacker News application with React and Redux. Hacker News is a platform to share news in and around the technology domain. It provides a [public API](https://hn.algolia.com/api) to interact with their data. Some of you might have read [the Road to learn React](https://www.robinwieruch.de/the-road-to-learn-react/) where you have build a Hacker News application as well. But that time it was only plain React. Now you can experience the differences when using Redux.

You are going to start with create-react-app to bootstrap your project. You can read the [official documentation](https://github.com/facebookincubator/create-react-app) to read about how it works. You simply start by choosing a project name for your application.

{title="Command Line",lang="text"}
~~~~~~~~
create-react-app react-redux-hackernews
~~~~~~~~

After the project was created for your, you can navigate into the project folder, open your editor and start the application.

{title="Command Line",lang="text"}
~~~~~~~~
cd react-redux-hackernews
npm start
~~~~~~~~

It should show the defaults that come with create-create-app.

## Part 1: Project Organization

Before you familiarze yourself with the folder structure in this part, you will adapt it to your own needs. First, move into the *src/* folder and delete the boilerplate files that are not needed for the application.

{title="Command Line: /",lang="text"}
~~~~~~~~
cd src
rm logo.svg App.js App.test.js App.css
~~~~~~~~

Even the `App` component gets deleted, because you'll organize it in folders instead of in the top level *src/* folder. Now, from the *src/* folder, create the folders for a organized folder structure by a technical separation.

{title="Command Line: src/",lang="text"}
~~~~~~~~
mkdir constants reducers actions selectors sagas components store
~~~~~~~~

Your folder structure should be similiar to the following:

{title="Folder Structure",lang="text"}
~~~~~~~~
-src/
--actions/
--api/
--components/
--constants/
--reducers/
--sagas/
--selectors/
--store/
--index.css
--index.js
~~~~~~~~

Navigate in the *component/* folder and create the following files for your independent components.

{title="Command Line: src/",lang="text"}
~~~~~~~~
cd components
touch index.js App.js Stories.js Story.js
~~~~~~~~

You can continue this way and create the remaining files to end up with the following folder structure.

{title="Folder Structure",lang="text"}
~~~~~~~~
-src/
--actions/
--api/
--components/
---App.js
---App.css
---Stories.js
---Stories.css
---Story.js
---Story.css
--constants/
---actionTypes.js
--reducers/
---index.js
---sagas/
----index.js
--selectors/
--store/
---index.js
--index.css
--index.js
~~~~~~~~

Now you have your foundation of folders and files for your React and Redux application. Except for the specific component files that you already have, everything else can be used as a blueprint, your own boilerplate, for any application using React and Redux. But only if it is separated by technical concerns. In a growing application you might want to separate your folders by feature.

## Part 2: Plain React Components

In this you will implement your plain React component architecture that only receives all necessary props from their parent components. These props can already have callback functions to do something. The point is that the props don't reveal that they are props themselves in the parent component, state from the local state or even Redux, or derived properties. The callback functions are plain functions too. Thus the components are not aware of using local state functions or Redux actions to alter the state.

In your entry point to React, where your root component gets rendered into the DOM, adjust the import of the `App` component by including the components folder in the path.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
# leanpub-start-insert
import App from './components/App';
# leanpub-end-insert
import './index.css';

ReactDOM.render(<App />, document.getElementById('root'));
~~~~~~~~

In the next step you can come up with sample data that can be used in the React components. The sample data becomes the input of the `App` component. At a later point in time this data will get fetched from the Hacker News API.

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const stories = [
  {
    title: 'React',
    url: 'https://facebook.github.io/react/',
    author: 'Jordan Walke',
    num_comments: 3,
    points: 4,
    objectID: 0,
  }, {
    title: 'Redux',
    url: 'https://github.com/reactjs/redux',
    author: 'Dan Abramov, Andrew Clark',
    num_comments: 2,
    points: 5,
    objectID: 1,
  },
];

ReactDOM.render(<App stories={stories} />, document.getElementById('root'));
# leanpub-end-insert
~~~~~~~~

The three components, `App`, `Stories` and `Story`, are not defined yet but you have already created the files. Let's define them component by component. First, the `App` receives the sample stories from above as props and its only responsiblity is to render the `Stories` component and to pass over the `stories` as props.

{title="src/components/App.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import './App.css';

import Stories from './Stories';

const App = ({ stories }) =>
  <div className="app">
    <Stories stories={stories} />
  </div>

export default App;
~~~~~~~~

Second, the `Stories` component receives the `stories` as props and renders for each story a `Story` component.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import './Stories.css';

import Story from './Story';

const Stories = ({ stories }) =>
  <div className="stories">
    {(stories || []).map(story =>
      <Story
        key={story.objectID}
        story={story}
      />
    )}
  </div>

export default Stories;
~~~~~~~~

Third, the `Story` component renders a few properties of the `story` object that gets destructured from the props object.

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import './Story.css';

const Story = ({ story }) => {
  const {
    title,
    url,
    author,
    num_comments,
    points,
  } = story;

  return (
    <div className="story">
      <span>
        <a href={url}>{title}</a>
      </span>
      <span>{author}</span>
      <span>{num_comments}</span>
      <span>{points}</span>
    </div>
  );
}

export default Story;
~~~~~~~~

You can start your application again with `npm start` on the command line. Both sample stories should be displayed in plain React.

## Part 3: Apply Styling

The application looks a bit dull without any styling. Therefore you can drop in styling some styling of your own or use the styling that's provided. First, the application needs some general style that can be defined in the root style file.

{title="src/index.css",lang="css"}
~~~~~~~~
body {
  color: #222;
  background: #f4f4f4;
  font: 400 14px CoreSans, Arial,sans-serif;
}

a {
  color: #222;
}

a:hover {
  text-decoration: underline;
}

ul, li {
  list-style: none;
  padding: 0;
  margin: 0;
}

input {
  padding: 10px;
  border-radius: 5px;
  outline: none;
  margin-right: 10px;
  border: 1px solid #dddddd;
}

button {
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #dddddd;
  background: transparent;
  color: #808080;
  cursor: pointer;
}

button:hover {
  color: #222;
}

.button-inline {
  border-width: 0;
  background: transparent;
  color: inherit;
  text-align: inherit;
  -webkit-font-smoothing: inherit;
  padding: 0;
  font-size: inherit;
  cursor: pointer;
}

.button-active {
  border-radius: 0;
  border-bottom: 1px solid #38BB6C;
}

*:focus {
  outline: none;
}
~~~~~~~~

Second, the `App` component gets a few CSS classes:

{title="src/components/App.css",lang="css"}
~~~~~~~~
.app {
  margin: 20px;
}

.interactions {
  text-align: center;
}
~~~~~~~~

Third, the `Stories` component gets some style:

{title="src/components/Stories.css",lang="css"}
~~~~~~~~
.stories {
  margin: 20px 0;
}

.stories-header {
  display: flex;
  line-height: 24px;
  font-size: 16px;
  padding: 0 10px;
  justify-content: space-between;
}

.stories-header > span {
  overflow: hidden;
  text-overflow: ellipsis;
  padding: 0 5px;
}
~~~~~~~~

And last but not least, the `Story` component will be painted:

{title="src/components/Story.css",lang="css"}
~~~~~~~~
.story {
  display: flex;
  line-height: 24px;
  white-space: nowrap;
  margin: 10px 0;
  padding: 10px;
  background: #ffffff;
  border: 1px solid #e3e3e3;
}

.story > span {
  overflow: hidden;
  text-overflow: ellipsis;
  padding: 0 5px;
}
~~~~~~~~

When you start your application again, it seems more organized by its styling. But there is still something missing for displaying the stories properly. The columns for each story should be aligned and perhaps there should be a heading for each column. First, you can define an object to describe the columns.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import './Stories.css';

import Story from './Story';

# leanpub-start-insert
const COLUMNS = {
  title: {
    label: 'Title',
    width: '40%',
  },
  author: {
    label: 'Author',
    width: '30%',
  },
  comments: {
    label: 'Comments',
    width: '10%',
  },
  points: {
    label: 'Points',
    width: '10%',
  },
  archive: {
    width: '10%',
  },
};
# leanpub-end-insert

const Stories = ({ stories }) =>
  ...
~~~~~~~~

The last column with the 'archive' property name is not used yet, but will be used in a later point in time. Second, you can pass this object to your `Story` component.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
const Stories = ({ stories }) =>
  <div className="stories">
    {(stories || []).map(story =>
      <Story
        key={story.objectID}
        story={story}
# leanpub-start-insert
        columns={COLUMNS}
# leanpub-end-insert
      />
    )}
  </div>
~~~~~~~~

The `Story` component can use it to style each displaying property of the story.

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const Story = ({ story, columns }) => {
# leanpub-end-insert

  ...

  return (
    <div className="story">
# leanpub-start-insert
      <span style={{ width: columns.title.width }}>
        <a href={url}>{title}</a>
      </span>
      <span style={{ width: columns.author.width }}>
        {author}
      </span>
      <span style={{ width: columns.comments.width }}>
        {num_comments}
      </span>
      <span style={{ width: columns.points.width }}>
        {points}
      </span>
      <span style={{ width: columns.archive.width }}>
      </span>
# leanpub-end-insert
    </div>
  );
}
~~~~~~~~

Last but not least, you can use the `COLUMNS` object to give your `Stories` component matching header columns. That's why the `COLUMNS` object got defined in the `Stories` component in the first place. Now, rather than doing it manually, as in the `Story` component, you will map the object dynamically to render the header columns. Since it is an object, you have to turn it into an array of the property names first, and then access the object by its mapped keys.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
const Stories = ({ stories }) =>
  <div className="stories">
# leanpub-start-insert
    <div className="stories-header">
      {Object.keys(COLUMNS).map(key =>
        <span
          key={key}
          style={{ width: COLUMNS[key].width }}
        >
          {COLUMNS[key].label}
        </span>
      )}
    </div>
# leanpub-end-insert

    {(stories || []).map(story =>
      <Story
        key={story.objectID}
        story={story}
        columns={COLUMNS}
      />
    )}
  </div>
~~~~~~~~

You can extract the header columns as its own `StoriesHeader` component to keep your component well arranged.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
const Stories = ({ stories }) =>
  <div className="stories">
# leanpub-start-insert
    <StoriesHeader columns={COLUMNS} />
# leanpub-end-insert

    {(stories || []).map(story =>
      ...
    )}
  </div>

# leanpub-start-insert
const StoriesHeader = ({ columns }) =>
  <div className="stories-header">
    {Object.keys(columns).map(key =>
      <span
        key={key}
        style={{ width: columns[key].width }}
      >
        {columns[key].label}
      </span>
    )}
  </div>
# leanpub-end-insert
~~~~~~~~

In this part you have painted your application and components with styling. The application should be in a representable state for none designers.

## Part 4: Archive a Story

Now you will add your first functionality: archiving a story. Therefore you will have to introduce Redux at some point to your application to manage the state of archived stories. I want to highly empahsize that it would work in plain React too. But for the sake of learning Redux, you will already use it at this point in time.

First, the archiving functionality can be passed down to the `Story` component from your React root component. In the beginning, it can be an empty function. The function will be replaced later.

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

ReactDOM.render(
# leanpub-start-insert
  <App stories={stories} onArchive={() => {}} />,
# leanpub-end-insert
  document.getElementById('root')
);
~~~~~~~~

Second, you can pass it through your `App` and `Stories` components. These components don't use the function but only pass it to the `Story` component. You might already notice that this could be a potential refactoring later on, because the function gets passed from the root component through a few components only to reach the leaf component.

{title="src/components/App.js",lang="javascript"}
~~~~~~~~
const App = ({ stories, onArchive }) =>
  <div className="app">
    <Stories
      stories={stories}
# leanpub-start-insert
      onArchive={onArchive}
# leanpub-end-insert
    />
  </div>
~~~~~~~~

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const Stories = ({ stories, onArchive }) =>
# leanpub-end-insert
  <div className="stories">
    <StoriesHeader columns={COLUMNS} />

    {(stories || []).map(story =>
      <Story
        key={story.objectID}
        story={story}
        columns={COLUMNS}
# leanpub-start-insert
        onArchive={onArchive}
# leanpub-end-insert
      />
    )}
  </div>
~~~~~~~~

Finally, you can use it in your `Story` component in a `onClick` handler of a button.

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const Story = ({ story, columns, onArchive }) => {
# leanpub-end-insert

  const {
    title,
    url,
    author,
    num_comments,
    points,
# leanpub-start-insert
    objectID,
# leanpub-end-insert
  } = story;

  return (
    <div className="story">
      ...
      <span style={{ width: columns.archive.width }}>
# leanpub-start-insert
        <button
          type="button"
          className="button-inline"
          onClick={() => onArchive(objectID)}
        >
          Archive
        </button>
# leanpub-end-insert
      </span>
    </div>
  );
}
~~~~~~~~

A refactoring that you could already do would be to extract the button as a reusable component.

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
const Story = ({ story, columns, onArchive }) => {
  ...

  return (
    <div className="story">
      ...
      <span style={{ width: columns.archive.width }}>
# leanpub-start-insert
        <ButtonInline onClick={() => onArchive(objectID)}>
          Archive
        </ButtonInline>
# leanpub-end-insert
      </span>
    </div>
  );
}

# leanpub-start-insert
const ButtonInline = ({
  onClick,
  type = 'button',
  children
}) =>
  <button
    type={type}
    className="button-inline"
    onClick={onClick}
  >
    {children}
  </button>
# leanpub-end-insert
~~~~~~~~

You can make even another more abstract `Button` component that doesn't share the `button-inline` CSS class.

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
...

const ButtonInline = ({
  onClick,
  type = 'button',
  children
}) =>
  <Button
    type={type}
    className="button-inline"
    onClick={onClick}
  >
    {children}
  </Button>

const Button = ({
  onClick,
  className,
  type = 'button',
  children
}) =>
  <button
    type={type}
    className={className}
    onClick={onClick}
  >
    {children}
  </button>
~~~~~~~~

Both button components should be extracted to a new file called *src/components/Buttons.js*, but exported so that at least the `ButtonInline` component can be reused in the `Story` component. Now, when you start your application again, the button to archive a story is there. But it doesn't work because it only receives a no-op (empty function) as property.

## Part 5: Introduce Redux: Store + First Reducer

This part will finally introduce Redux to manage the state of the (sample) stories instead of passing it directly into your component tree. Let's approach it step by step. First, you have to install Redux on the command line:

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save redux
~~~~~~~~

Second, in the root entry point of React, you can import the Redux store. The store is not defined yet. Instead of using the sample stories, you will use the stories that are stored in the Redux store. Taken that the store saves only a list of stories as state, you can simply get the root state of the store and assume that it is the list of stories.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
# leanpub-start-insert
import store from './store';
# leanpub-end-insert
import './index.css';

ReactDOM.render(
# leanpub-start-insert
  <App stories={store.getState()} />,
# leanpub-end-insert
  document.getElementById('root')
);
~~~~~~~~

Third, you have to create your Redux store instance in a separate file. It already takes a reducer that is not implemented yet. You will implement it in the next step.

{title="src/store/index.js",lang="javascript"}
~~~~~~~~
import { createStore } from 'redux';
import storyReducer from '../reducers/story';

const store = createStore(
  storyReducer
);

export default store;
~~~~~~~~

Fourth, in your *src/reducers/* folder you can create your first reducer: `storyReducer`. It can have the sample stories as initial state .

{title="src/reducers/story.js",lang="javascript"}
~~~~~~~~
const INITIAL_STATE = [
  {
    title: 'React',
    url: 'https://facebook.github.io/react/',
    author: 'Jordan Walke',
    num_comments: 3,
    points: 4,
    objectID: 0,
  }, {
    title: 'Redux',
    url: 'https://github.com/reactjs/redux',
    author: 'Dan Abramov, Andrew Clark',
    num_comments: 2,
    points: 5,
    objectID: 1,
  },
];

function storyReducer(state = INITIAL_STATE, action) {
  switch(action.type) {
    default : return state;
  }
}

export default storyReducer;
~~~~~~~~

Your application should work when you start it. It is using the Redux store to retrieve the initial state from the `storyReducer`, because it is the only reducer in your application. There are no actions yet and no action is captured in the reducer yet. Even though there was no action dispatched yet, you can see that the Redux store runs through all its defined reducers to initialize its initial state in the store. The state gets visible through the `Stories` and `Story` components, because it is passed down from the React root entry point.

## Part 7: Two Reducers

You have used the Redux store to define an initial state of sample stories and to retrieve this state for your component tree. But there is no state manipulation happening yet. In this part and the next part you are going to implement the archive functionality. When approaching this functionality, the simplest thing to do would be to remove the archived story from the list of stories in the `storyReducer`. But let's approach this from a different angle to have a greater impact in the long run. It could still be useful to have all stories in the end, but have a way to distinguish between them: stories and archived stories. Following this way, you would be able in the future to have a second component that shows the archived stories next to the available stories.

From an implementation point of view, the `storyReducer` will stay as it is for now. But you can introduce a second reducer, a `archiveReducer`, that keeps a list of references to the archived stories.

{title="src/reducers/archive.js",lang="javascript"}
~~~~~~~~
const INITIAL_STATE = [];

function archiveReducer(state = INITIAL_STATE, action) {
  switch(action.type) {
    default : return state;
  }
}

export default archiveReducer;
~~~~~~~~

You will implement the action to archive a story in a second. First, the Redux store in its instantation needs to get both reducers now. It has to get the combined reducer. Let's pretend that the store can import the combined reducer from the entry file, the *reducers/index.js*, without worrying about the combining of the reducers yet.

{title="src/store/index.js",lang="javascript"}
~~~~~~~~
import { createStore } from 'redux';
# leanpub-start-insert
import rootReducer from '../reducers';
# leanpub-end-insert

const store = createStore(
# leanpub-start-insert
  rootReducer
# leanpub-end-insert
);

export default store;
~~~~~~~~

Next you can combine both reducers in the file that is used by the Redux store to import the `rootReducer`.

{title="src/reducers/index.js",lang="javascript"}
~~~~~~~~
import { combineReducers } from 'redux';
import storyReducer from './story';
import archiveReducer from './archive';

const rootReducer = combineReducers({
  storyState: storyReducer,
  archiveState: archiveReducer,
});

export default rootReducer;
~~~~~~~~

Since your state is sliced up into two substates now, you have to adjust how you retrieve the stories from your store with the intermediate `storyState`. This is a crucial step, because it shows how a combined reducer slices up your state into substates.

{title="src/index.js",lang="javascript"}
~~~~~~~~
ReactDOM.render(
  <App
# leanpub-start-insert
    stories={store.getState().storyState}
# leanpub-end-insert
    onArchive={() => {}}
  />,
  document.getElementById('root')
);
~~~~~~~~

The application should show up the same stories as before when you start it. However, there is still no state manipulation happening, because no actions are involved yet. Finally in the next part you will dispatch your first action to archive a story.

## Part 8: First Action

In this part you will dispatch your first action to archive a story. The archive action needs to be captured in the new `archiveReducer`. It simply stores all archived stories by their id in a list. The initial state is an empty list, because no story is archived in the beginning.

{title="src/reducers/archive.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { STORY_ARCHIVE } from '../constants/actionTypes';
# leanpub-end-insert

const INITIAL_STATE = [];

# leanpub-start-insert
const applyArchiveStory = (state, action) =>
  [ ...state, action.id ];
# leanpub-end-insert

function archiveReducer(state = INITIAL_STATE, action) {
  switch(action.type) {
# leanpub-start-insert
    case STORY_ARCHIVE : {
      return applyArchiveStory(state, action);
    }
# leanpub-end-insert
    default : return state;
  }
}

export default archiveReducer;
~~~~~~~~

The action type is already outsourced in a different file. This way it can be reused when dispatching the action from the Redux store.

{title="src/constants/actionTypes.js",lang="javascript"}
~~~~~~~~
export const STORY_ARCHIVE = 'STORY_ARCHIVE';
~~~~~~~~

Last but not least, you can import the action type and dispatch the whole action in your root component.

{title="src/reducers/archive.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
import store from './store';
# leanpub-start-insert
import { STORY_ARCHIVE } from './constants/actionTypes';
# leanpub-end-insert
import './index.css';

ReactDOM.render(
  <App
    stories={store.getState().storyState}
# leanpub-start-insert
    onArchive={id => store.dispatch({ type: STORY_ARCHIVE, id })}
# leanpub-end-insert
  />,
  document.getElementById('root')
);
~~~~~~~~

Now you dispatch the action directly without an action creator. When you start your application, it shoudl still work, but nothing happens when archiving a story. The archived stories are not yet evaluated in the component tree.

## Part 9: First Selector

You can use both substates, `storyState` and `archiveState` to derive the lsit of stories that are not archiveed. The deriving of those properties can happen in a selector. You can create your first selector that only returns the part of the stories that is not archived.

{title="src/selectors/story.js",lang="javascript"}
~~~~~~~~
const isNotArchived = archivedIds => story =>
  archivedIds.indexOf(story.objectID) === -1;

const getReadableStories = ({ storyState, archiveState }) =>
  storyState.filter(isNotArchived(archiveState));

export {
  getReadableStories,
};
~~~~~~~~

The selector makes heaviliy use of JavaScript ES6 arrow functions, JavaScript ES6 destructuring and a higher order function: `isNotArchived()`. If you are not used to JavaScript ES6, don't feel intimidated by it. It is only a way to express these functions more concise in a functional programmign style. In plain JavaScript ES5 it would look like the following:

{title="src/selectors/story.js",lang="javascript"}
~~~~~~~~
function isNotArchived(archivedIds) {
  return function (story) {
    return archivedIds.indexOf(story.objectID) === -1;
  };
}

function getReadableStories({ storyState, archiveState }) {
  return storyState.filter(isNotArchived(archiveState));
}

export {
  getReadableStories,
};
~~~~~~~~

Last but not least, you can use the selector to compute the not archived stories instead of retrieving the whole list of stories from the store directly.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
import store from './store';
# leanpub-start-insert
import { getReadableStories } from './selectors/story';
# leanpub-end-insert
import { STORY_ARCHIVE } from './constants/actionTypes';
import './index.css';

ReactDOM.render(
  <App
# leanpub-start-insert
    stories={getReadableStories(store.getState())}
# leanpub-end-insert
    onArchive={id => store.dispatch({ type: STORY_ARCHIVE, id })}
  />,
  document.getElementById('root')
);
~~~~~~~~

When you start your application, nothing happens when you archive a story. There is no re-rendering of the view in place to update it.

## Part 11: Re-render View

In this part you will update the view layer to reflect the correct state that is used from the Redux store. When an action dispatches, the state in the Redux store gets updated. However, the component tree in React doesn't update, because no one subscribed to the Redux store yet. In the first attempt, you are going to wire up Redux and React naively and re-render the whole component tree on each update.

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
function render() {
# leanpub-end-insert
  ReactDOM.render(
    <App
      stories={getReadableStories(store.getState())}
      onArchive={id => store.dispatch({ type: STORY_ARCHIVE, id })}
    />,
    document.getElementById('root')
  );
# leanpub-start-insert
}
# leanpub-end-insert

# leanpub-start-insert
store.subscribe(render);
render();
# leanpub-end-insert
~~~~~~~~

Now the components will re-render once you archive a story, because the state in the Redux store updates and the subscription will run to render again the whole component tree. Congratulations, you dispatched your first action, selected derived properties from the state and updated your component tree by subscribing it to the Redux store. That took longer as expected, didn't it? However, now most of the Redux and React infrastructure is in place to be more efficient when introducing new features.

## Part 12: First Middleware

In this chapter you will introduce your first middleware to the Redux store. In a scaling application it becomes often a problem to track state updates. Often you don't notice when an action is dispatched, because too many actions get involved and a bunch of them might get triggered implictly. Therefore you can use the [redux-logger](https://github.com/evgenyrodionov/redux-logger) middleware in your Redux store to `console.log()` every action, the previous state and the next state, automarically to your developers console when dispatching an action. First, you have to install the neat middleware library.

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save redux-logger
~~~~~~~~

Second, you can use it as middleware in your Redux store initialization.

{title="src/store.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { createStore, applyMiddleware } from 'redux';
import { createLogger } from 'redux-logger';
# leanpub-end-insert
import rootReducer from '../reducers';

# leanpub-start-insert
const logger = createLogger();
# leanpub-end-insert

const store = createStore(
  rootReducer,
# leanpub-start-insert
  undefined,
  applyMiddleware(logger)
# leanpub-end-insert
);

export default store;
~~~~~~~~

That's it. Every time you dispatch an action now, for instance when archiving a story, you will see the logging in the developer console in your browser.

## Part 13: First Action Creator

The action you are dispatching is a plain action object. However, you might want to reuse it in a later point in time. Action creators are not mandatory, but they keep your Redux architecture organized. In order to stay organized, let's define the first action creator. First, you have to define the action creator that takes a story id, to identify the archiving story, in a new file.

{title="src/actions/archive.js",lang="javascript"}
~~~~~~~~
import { STORY_ARCHIVE } from '../constants/actionTypes';

const doArchiveStory = id => ({
  type: STORY_ARCHIVE,
  id,
});

export {
  doArchiveStory,
};
~~~~~~~~

Second, you can use it in your root component. Instead of dispatchign the action object directly, you can create an action by using its action creator.

{title="src/actions/archive.js",lang="javascript"}
~~~~~~~~
...
# leanpub-start-insert
import { doArchiveStory } from './actions/archive';
# leanpub-end-insert

function render() {
  ReactDOM.render(
    <App
      stories={getReadableStories(store.getState())}
# leanpub-start-insert
      onArchive={id => store.dispatch(doArchiveStory(id))}
# leanpub-end-insert
    />,
    document.getElementById('root')
  );
}

...
~~~~~~~~

The application should operate as before.

## Part 14: Connect React with Redux

In this part you will connect the React and Redux in a more sophisticated way. The component tree already re-renders when you dispatch an action. However, you might  want to wire up components indepdently with the Redux store without using the Redux store directly. In addition, you don't want to re-render the whole component tree, but only the components where the state or props have changed. Let's change this by using the react-redux library that connects both worlds.

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save react-redux
~~~~~~~~

You can use the `Provider` component, which makes the Redux store available to all components below, in your React root entry point.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
# leanpub-start-insert
import { Provider } from 'react-redux';
# leanpub-end-insert
import App from './components/App';
import store from './store';
import './index.css';

# leanpub-start-insert
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
# leanpub-end-insert
~~~~~~~~

Notice that the render method isn't used in a Redux store subscription anymore. No one subscribes to the Redux store and the `App` component isn't receiving any props. In addition, the `App` component is only rendering component and doesn't pass any props.

{title="src/components/App.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import './App.css';

import Stories from './Stories';

const App = () =>
  <div className="app">
# leanpub-start-insert
    <Stories />
# leanpub-end-insert
  </div>

export default App;
~~~~~~~~

But who gives the props to the `Stories` component? This component is the first component that needs to know about the list of stories, because it has to display it. The solution is to upgrade the `Stories` component to a connected component. So, instead of only exporting the plain `Stories` component:

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
...

export default Stories;
~~~~~~~~

You can export the connected component that has access to the store:

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { connect } from 'react-redux';
import { doArchiveStory } from '../actions/archive';
import { getReadableStories } from '../selectors/story';
# leanpub-end-insert

...

# leanpub-start-insert
const mapStateToProps = state => ({
  stories: getReadableStories(state),
});

const mapDispatchToProps = dispatch => ({
  onArchive: id => dispatch(doArchiveStory(id)),
});

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Stories);
# leanpub-end-insert
~~~~~~~~

The `Stories` component is a connected component now and is the only component that has access to the Redux store. The application work again, but this time with a clever interaction between Redux and React.

## Part 15: Lift Connection

It is no official term, but you can lift the connection between React and Redux. For instance, you could lift the connection from the `Stories` component to another component. But you need the list of stories to map over them in the `Stories` component. However, what about the `onArchive` function? It is not used in the `Stories` component, but only in the `Story` component. Thus you could lift the connection partly. The `stories` would stay in the `Stories` component, but the `onArchive` function could live in the `Story` component.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const Stories = ({ stories }) =>
# leanpub-end-insert
  <div className="stories">
    <StoriesHeader columns={COLUMNS} />

    {(stories || []).map(story =>
      <Story
        key={story.objectID}
        story={story}
        columns={COLUMNS}
      />
    )}
  </div>

...

const mapStateToProps = state => ({
  stories: getReadableStories(state),
});

# leanpub-start-insert
export default connect(
  mapStateToProps
)(Stories);
# leanpub-end-insert
~~~~~~~~

Instead you can connect the `Story` component now. You would have two connected components afterward.

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { connect } from 'react-redux';
import { doArchiveStory } from '../actions/archive';
# leanpub-end-insert

...

# leanpub-start-insert
const mapDispatchToProps = dispatch => ({
  onArchive: id => dispatch(doArchiveStory(id)),
});

export default connect(
  null,
  mapDispatchToProps
)(Story);
# leanpub-end-insert
~~~~~~~~

With this refactoring step in your mind, you can always lift your connections to the Redux store from your view layer depending on the needs of the components. Does the component need state from the Redux store? Does the component need to alter the state in the Redux store via dispatching an action? You are in full control of where you want to use connected components and where you want to keep your components as presenter components.

## Part 16: Interacting with an API

Implementing applications with sample data can be dull. It can be more exciting by interacting with a real API - the [Hacker News API](https://hn.algolia.com/api). Even though, as you have learned, you can have asynchronous actions without another library, this application will introduce Redux Saga as asynchrnours action library to deal with side-effects such as fetching data from a third-party library.

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save redux-saga
~~~~~~~~

First, you can introduce a root saga in your entry point file to sagas. It can be similar seen to the combined root reducer, because in the end the Redux store expects one reducer or one saga for its creation.

{title="src/sagas/index.js",lang="javascript"}
~~~~~~~~
import { takeEvery } from 'redux-saga/effects';
import { STORIES_FETCH } from '../constants/actionTypes';
import { handleFetchStories } from './story';

function *watchAll() {
  yield [
    takeEvery(STORIES_FETCH, handleFetchStories),
  ]
}

export default watchAll;
~~~~~~~~

Second, the root saga can be used in the Redux store middleware when initializing the saga middleware.

{title="src/sagas/index.js",lang="javascript"}
~~~~~~~~
import { createStore, applyMiddleware } from 'redux';
import { createLogger } from 'redux-logger';
# leanpub-start-insert
import createSagaMiddleware from 'redux-saga';
# leanpub-end-insert
import rootReducer from '../reducers';
# leanpub-start-insert
import rootSaga from '../sagas';
# leanpub-end-insert

const logger = createLogger();
# leanpub-start-insert
const saga = createSagaMiddleware();
# leanpub-end-insert

const store = createStore(
  rootReducer,
  undefined,
# leanpub-start-insert
  applyMiddleware(saga, logger)
# leanpub-end-insert
);

# leanpub-start-insert
saga.run(rootSaga);
# leanpub-end-insert

export default store;
~~~~~~~~

Third, you can introduce the new action type in your constants that will trigger the saga. However, you can already introduce a second action type that will later on - when the request succeeds - add the stories in your `storyReducer` to the Redux store. Basically you have one action to trigger the side-effect that is handled with Redux Saga and one action that stores the result of the side-effect in the Redux store.

{title="src/constants/actionTypes.js",lang="javascript"}
~~~~~~~~
export const STORY_ARCHIVE = 'STORY_ARCHIVE';
# leanpub-start-insert
export const STORIES_FETCH = 'STORIES_FETCH';
export const STORIES_ADD = 'STORIES_ADD';
# leanpub-end-insert
~~~~~~~~

Fourth, you can implement the story saga that encapsulates the API request

{title="src/sagas/story.js",lang="javascript"}
~~~~~~~~
import { call, put } from 'redux-saga/effects';
import { doAddStories } from '../actions/story';

const HN_BASE_URL = 'http://hn.algolia.com/api/v1/search?query=';

const fetchStories = query =>
  fetch(HN_BASE_URL + query)
    .then(response => response.json());

function* handleFetchStories(action) {
  const { query } = action;
  const result = yield call(fetchStories, query);
  yield put(doAddStories(result.hits));
}

export {
  handleFetchStories,
};
~~~~~~~~

In the fifth step, you need to define both actions creators: the first one that triggers the side-effect to fetch stories by a search term and the second one that adds the fetched stories to your Redux store.

{title="src/actions/story.js",lang="javascript"}
~~~~~~~~
import {
  STORIES_ADD,
  STORIES_FETCH,
} from '../constants/actionTypes';

const doAddStories = stories => ({
  type: STORIES_ADD,
  stories,
});

const doFetchStories = query => ({
  type: STORIES_FETCH,
  query,
});

export {
  doAddStories,
  doFetchStories,
};
~~~~~~~~

Only the second action needs to be intercepted in your `storyReducer` to store the stories. The first action is only used to trigger the Redux Saga. Don't forget to remove the sample stories.

{title="src/reducers/story.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { STORIES_ADD } from '../constants/actionTypes';

const INITIAL_STATE = [];

const applyAddStories = (state, action) =>
  [ ...action.stories ];
# leanpub-end-insert

function storyReducer(state = INITIAL_STATE, action) {
  switch(action.type) {
# leanpub-start-insert
    case STORIES_ADD : {
      return applyAddStories(state, action);
    }
# leanpub-end-insert
    default : return state;
  }
}

export default storyReducer;
~~~~~~~~

Now, everything is setup from a Redux and Redux Saga perspective. As last step, only one component from the view layer needs to trigger the `STORIES_FETCH` action that is intercepted in the saga, fetches the stories in a side-effect, and stores them in the Redux store with the `STORIES_ADD` action. Therefore, in your `App` component, you can introduce the new `SearchStories` component.

{title="src/reducers/story.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import './App.css';

import Stories from './Stories';
# leanpub-start-insert
import SearchStories from './SearchStories';
# leanpub-end-insert

const App = () =>
  <div className="app">
# leanpub-start-insert
    <div className="interactions">
      <SearchStories />
    </div>
# leanpub-end-insert
    <Stories />
  </div>

export default App;
~~~~~~~~

The `SearchStories` component will be a connected component. The next step is to implement that component. First, you start with a plain React component that has a form, input field and button.

{title="src/components/SearchStories.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
import Button from './Buttons';

class SearchStories extends Component {
  constructor(props) {
    super(props);

    this.state = {
      query: '',
    };
  }

  render() {
    return (
      <form onSubmit={this.onSubmit}>
        <input
          type="text"
          value={this.state.query}
          onChange={this.onChange}
        />
        <Button type="submit">
          Search
        </Button>
      </form>
    );
  }
}

export default SearchStories;
~~~~~~~~

There are two class methods you would have to introduce for the `SearchStories` component to make it work.

{title="src/components/SearchStories.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const applyQueryState = query => () => ({
  query
});
# leanpub-end-insert

class SearchStories extends Component {
  constructor(props) {
    ...

# leanpub-start-insert
    this.onChange = this.onChange.bind(this);
    this.onSubmit = this.onSubmit.bind(this);
# leanpub-end-insert
  }

# leanpub-start-insert
  onSubmit(event) {
    const { query } = this.state;
    if (query) {
      // TODO: not defined yet
      this.props.onFetchStories(query)

      this.setState(applyQueryState(''));
    }

    event.preventDefault();
  }

  onChange(event) {
    const { value } = event.target;
    this.setState(applyQueryState(value));
  }
# leanpub-end-insert

  render() {
    ...
  }
}

export default SearchStories;
~~~~~~~~

The component should work on its own now. It only receives one function from the outside via its props. This function will dispatch an action to trigger the saga that fetches the stories from the Hacker News platform. You would have to connect the `SearchStories` component to make the dispatch functionality available.

{title="src/components/SearchStories.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
# leanpub-start-insert
import { connect } from 'react-redux';
import { doFetchStories } from '../actions/story';
# leanpub-end-insert
import Button from './Buttons';

...

# leanpub-start-insert
const mapDispatchToProps = (dispatch) => ({
  onFetchStories: query => dispatch(doFetchStories(query)),
});

export default connect(
  null,
  mapDispatchToProps
)(SearchStories);
# leanpub-end-insert
~~~~~~~~

Start your application again and try to search for stories such as "React" or "Redux". It should work now. The connect component dispatches an action that triggers the saga. The side-effect of the saga is the fetching process of the stories by search term from the Hacker News API. Once the request succeeds, another actions get dispatched and captured in the `storyReducer` to finally store the stories.

## Part 17: Separation of API

There is one last refactoring step that you could apply. It would improve the separation between API functionalities and sagas. You would extract the API call from the story saga into an own API folder. Afterward, other sagas could make use of these API requests too. First, extract the functionality from the saga:

{title="src/sagas/story.js",lang="javascript"}
~~~~~~~~
import { call, put } from 'redux-saga/effects';
import { doAddStories } from '../actions/story';
# leanpub-start-insert
import { fetchStories } from '../api/story';
# leanpub-end-insert

function* handleFetchStories(action) {
  const { query } = action;
  const result = yield call(fetchStories, query);
  yield put(doAddStories(result.hits));
}

export {
  handleFetchStories,
};
~~~~~~~~

And second, use it in an own dedicated API file.

{title="src/api/story.js",lang="javascript"}
~~~~~~~~
const HN_BASE_URL = 'http://hn.algolia.com/api/v1/search?query=';

const fetchStories = query =>
  fetch(HN_BASE_URL + query)
    .then(response => response.json());

export {
  fetchStories,
};
~~~~~~~~

Great, you have separated the API functionality from the saga.

## Final Words

Implementing this application could go on infinetely. I would have plenty of features in my head that I would want to add to it. What about you? Can you imagine to continue building this application? From a technical perspective, things that were taught in this book, everything is set up to give you the perfect starting point. However, there were more topics that you could apply. For instance, you could normalize your incoming stories from the API before they reach the Redux store. The following list should give you an idea about potential next steps:

* Normalize: The data that comes from the Hacker News API could be noamrlized before it reaches the reducer and finally the Redux store. You could use the library normlaizr that was introduced earlier in the book.

* React Router: All archived stories are captured in a separated substate. No stories gets removed yet from the Redux store. Thus you could use this archived stories to display it in another component. One step furhter could be to introduce React Router to manage two routes: the stories search result and the archived stories.

* Paginated Data: The response from the Hacker News API doesn't only return the list of stories. It returns a paginated list of stories with a page property. You could use the page property for fetch more stories with the same search term. The list component in React could be a [paginated list](https://www.robinwieruch.de/react-paginated-list/) or [infinite scroll list](https://www.robinwieruch.de/react-infinite-scroll/).

* Error Handling: There was no error handling in place yet. But you could introduce it with your sagas by capturing the errors and storing them in your Redux store. Afterward, you can communicate the error with a proper message in one of your components.

* Test: There are no tests yet for your state handling. You could introduce at least tests for your reducers, to ensure that the correct state is saved in the Redux store.

- archived in local storage
- caching