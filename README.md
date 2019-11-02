# react-storybook-example
This project shows you how to quickly create a react application and setup storybook. Additionally, it goes into the core functionality 

## Setup React & Storybook

```
# Create our application:
npx create-react-app taskbox
cd taskbox

# Add Storybook:
npx -p @storybook/cli sb init
```

```
# Run Application
yarn start

# Run Storybook alongside app
yarn storybook
```

Add some styling from this

## Create a Simple Component

Create the initial component. It takes in a simple function with props and actions. It should return basic HTML for this example.

```
// src/components/Task.js

import React from 'react';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  return (
    <div className="list-item">
      <input type="text" value={title} readOnly={true} />
    </div>
  );
}
```

Now we can create a few stories for this component. Start with importing the component. once you have that, we will need to define some object data to be able to pass in to the component. The actions are a Storybook addon that we will be talking about later.

Using `storiesOf`, we can create 3 different implementations of the same component.

```
// src/components/Task.stories.js

import React from 'react';
import { storiesOf } from '@storybook/react';
import { action } from '@storybook/addon-actions';

import Task from './Task';

export const task = {
  id: '1',
  title: 'Test Task',
  state: 'TASK_INBOX',
  updatedAt: new Date(2018, 0, 1, 9, 0),
};

export const actions = {
  onPinTask: action('onPinTask'),
  onArchiveTask: action('onArchiveTask'),
};

storiesOf('Task', module)
  .add('default', () => <Task task={task} {...actions} />)
  .add('pinned', () => <Task task={{ ...task, state: 'TASK_PINNED' }} {...actions} />)
  .add('archived', () => <Task task={{ ...task, state: 'TASK_ARCHIVED' }} {...actions} />);
```

Looks great! But, we can't see the component in storybook yet. We need to update some settings for Storbook first.

```
import { configure } from '@storybook/react';
import '../src/index.css';

const req = require.context('../src', true, /\.stories.js$/);

function loadStories() {
  req.keys().forEach(filename => req(filename));
}

configure(loadStories, module);
```

Now we should be able to see our first items displayed! The do not currently look different, but that will change once we build out the states.

```
// src/components/Task.js

import React from 'react';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  return (
    <div className={`list-item ${state}`}>
      <label className="checkbox">
        <input
          type="checkbox"
          defaultChecked={state === 'TASK_ARCHIVED'}
          disabled={true}
          name="checked"
        />
        <span className="checkbox-custom" onClick={() => onArchiveTask(id)} />
      </label>
      <div className="title">
        <input type="text" value={title} readOnly={true} placeholder="Input title" />
      </div>

      <div className="actions" onClick={event => event.stopPropagation()}>
        {state !== 'TASK_ARCHIVED' && (
          <a onClick={() => onPinTask(id)}>
            <span className={`icon-star`} />
          </a>
        )}
      </div>
    </div>
  );
}
```

The last thing we need to do is specify some data requirements. This makes development easier by documenting how the data should look and hopefully find issues before you go to build your application.

## Combining Components

Add TaskList & TaskList stories to components folder to bring together all the individual items.

```
// src/components/TaskList.js

import React from 'react';

import Task from './Task';

function TaskList({ loading, tasks, onPinTask, onArchiveTask }) {
  const events = {
    onPinTask,
    onArchiveTask,
  };

  if (loading) {
    return <div className="list-items">loading</div>;
  }

  if (tasks.length === 0) {
    return <div className="list-items">empty</div>;
  }

  return (
    <div className="list-items">
      {tasks.map(task => (
        <Task key={task.id} task={task} {...events} />
      ))}
    </div>
  );
}

export default TaskList;
```

```
// src/components/TaskList.stories.js

import React from 'react';
import { storiesOf } from '@storybook/react';

import TaskList from './TaskList';
import { task, actions } from './Task.stories';

export const defaultTasks = [
  { ...task, id: '1', title: 'Task 1' },
  { ...task, id: '2', title: 'Task 2' },
  { ...task, id: '3', title: 'Task 3' },
  { ...task, id: '4', title: 'Task 4' },
  { ...task, id: '5', title: 'Task 5' },
  { ...task, id: '6', title: 'Task 6' },
];

export const withPinnedTasks = [
  ...defaultTasks.slice(0, 5),
  { id: '6', title: 'Task 6 (pinned)', state: 'TASK_PINNED' },
];

storiesOf('TaskList', module)
  .addDecorator(story => <div style={{ padding: '3rem' }}>{story()}</div>)
  .add('default', () => <TaskList tasks={defaultTasks} {...actions} />)
  .add('withPinnedTasks', () => <TaskList tasks={withPinnedTasks} {...actions} />)
  .add('loading', () => <TaskList loading tasks={[]} {...actions} />)
  .add('empty', () => <TaskList tasks={[]} {...actions} />);
```

The `addDecorator` call allows us to nest the component in HTML. In this case, it adds some nice padding around the data.

Just like we did with the individual tasks, we can create different dynamic states for the task list.

```
	// src/components/TaskList.js

	import React from 'react';

	import Task from './Task';

	function TaskList({ loading, tasks, onPinTask, onArchiveTask }) {
		const events = {
			onPinTask,
			onArchiveTask,
		};

		const LoadingRow = (
			<div className="loading-item">
				<span className="glow-checkbox" />
				<span className="glow-text">
					<span>Loading</span> <span>cool</span> <span>state</span>
				</span>
			</div>
		);

		if (loading) {
			return (
				<div className="list-items">
					{LoadingRow}
					{LoadingRow}
					{LoadingRow}
					{LoadingRow}
					{LoadingRow}
					{LoadingRow}
				</div>
			);
		}

		if (tasks.length === 0) {
			return (
				<div className="list-items">
					<div className="wrapper-message">
						<span className="icon-check" />
						<div className="title-message">You have no tasks</div>
						<div className="subtitle-message">Sit back and relax</div>
					</div>
				</div>
			);
		}

		const tasksInOrder = [
			...tasks.filter(t => t.state === 'TASK_PINNED'),
			...tasks.filter(t => t.state !== 'TASK_PINNED'),
		];

		return (
			<div className="list-items">
				{tasksInOrder.map(task => (
					<Task key={task.id} task={task} {...events} />
				))}
			</div>
		);
	}

	export default TaskList;
```

And just like we did with the tasks, we can setup propTypes to ensure the data is standardized.

```
// src/components/TaskList.js

TaskList.propTypes = {
  loading: PropTypes.bool,
  tasks: PropTypes.arrayOf(Task.propTypes.task).isRequired,
  onPinTask: PropTypes.func.isRequired,
  onArchiveTask: PropTypes.func.isRequired,
};

TaskList.defaultProps = {
  loading: false,
};

export default TaskList;
```

## Using Addons

Let's utilize an awesome feature of Storybook, Addons!

`yarn add @storybook/addon-knobs`

```
// .storybook/addons.js

import '@storybook/addon-actions/register';
import '@storybook/addon-knobs/register';
import '@storybook/addon-links/register';
```

Now that we have the knobs addon, let's use it!

```
// src/components/Task.stories.js

import React from 'react';
import { storiesOf } from '@storybook/react';
import { action } from '@storybook/addon-actions';
import { withKnobs, object } from '@storybook/addon-knobs/react';
```

```
// src/components/Task.stories.js

storiesOf('Task', module)
  .addDecorator(withKnobs)
  .add('default', () => {
    return <Task task={object('task', { ...task })} {...actions} />;
  })
```
