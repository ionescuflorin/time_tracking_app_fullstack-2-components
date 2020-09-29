# CH2 - COMPONENTS
---

## 1. A time logging app - Intro
- We investigate a pattern that you can use to build React apps from scratch and then put those steps to work to build an interface for managing timers.

#### #1.1. BREAKING THE APP INTO COMPONENTS

![app](ss/1.png)

- However, there’s one **subtle difference**: This list of timers has a little **“+” icon** at the bottom. As we saw, we’re able to add new timers to the list using this button.
- So, in reality, the TimerList component isn’t just a list of timers. **It also contains a widget** to create new timers.
- A component should, ideally, **only be responsible for one piece of functionality**
- So, the proper response here is for us to **shrink TimerList back into its responsibility of just listing timers and to nest it under a parent component.** 
- We’ll call the parent **TimersDashboard**. TimersDashboard **will have TimerList** and the **“+”/create form widget as children**:

![app](ss/2.png)
- The “+”/create form widget is interesting because it has two distinct representations. When the “+” button is clicked, the widget transmutes into a form. When the form is closed, the widget transmutes back into a “+” button.
- We’ll call it **ToggleableTimerForm**. As a child, it can either render the component TimerForm or the HTML markup for the “+” button.
---
- The timer itself has a fair bit of functionality. It can **transform into an edit form, delete itself, and start and stop itself**. 
- Do we need to break this up? And if so, how?
- **Displaying a timer and editing a timer are indeed two distinct UI elements**. 
- We need some container component that **renders either the timer’s face or its edit form depending on if the timer is being edited**.
- We’ll call this **EditableTimer**. 
- The c**hild of EditableTimer** will then be either **a Timer component** or **the edit form component**. 
- The form for creating and editing timers is very similar, so let’s assume that we can use the component TimerForm in both contexts:

![app](ss/3.png)

---
- So, we have our final component hierarchy, with some ambiguity around the final
state of the timer component:

![app](ss/4.png)


- **TimersDashboard**: Parent container
  - **EditableTimerList**: Displays a list of timer containers
    - **EditableTimer**: Displays either a timer or a timer’s edit form 
      - **Timer**: Displays a given timer
      - **TimerForm**: Displays a given timer’s edit form
  - **ToggleableTimerForm**: Displays a form to create a new timer
    - **TimerForm** (not displayed): Displays a new timer’s create form


- Represented as a hierarchical tree:

![app](ss/5.png)

---
  
## 2. The steps for building React apps from scratch
- Ultimately, our **top-level component** will **communicate** with a **server**. 
- The **server will be the initial source of state**, and React will render itself according to the data the server provides. 
- Our app will also **send updates to the server**, *like when a timer is started*:

![app](ss/6.png)

### !!React framework for developing a React app from scratch:
1. Break the app into components
2. Build a static version of the app
3. Determine what should be stateful
4. Determine in which component each piece of state should live 5. Hard-code initial states
6. Add inverse data flow
7. Add server communication

- We’ve already covered **step (1)** and **have a good understanding of all of our components**, save for some uncertainty down at the Timer component.
---

#### #2.1. [STEP 2]: BUILD A STATIC VERSION OF THE APP

##### TimersDashboard
```javascript
class TimersDashboard extends React.Component {
    render() {
      return (
        <div className='ui three column centered grid'>
          <div className='column'>
            <EditableTimerList />
            <ToggleableTimerForm
              isOpen={true}
            />
          </div>
        </div>
      );
    }
  }
```

- this component renders two child components:
  - **EditableTimerList**
  - **ToggleableTimerForm**
    - prop **isOpen={true}** is used by the child component to determine whether to render a **“+”** or **TimerForm**.
              

##### EditableTimerList
```javascript
  class EditableTimerList extends React.Component {
    render() {
      return (
        <div id='timers'>
          <EditableTimer
            title='Learn React'
            project='Web Domination'
            elapsed='8986300'
            runningSince={null}
            editFormOpen={false}
          />
          <EditableTimer
            title='Learn extreme ironing'
            project='World Domination'
            elapsed='3890985'
            runningSince={null}
            editFormOpen={true}
          />
        </div>
      );
    }
  }
```

##### EditableTimer
```javascript
  class EditableTimer extends React.Component {
    render() {
      if (this.props.editFormOpen) {
        return (
          <TimerForm
            title={this.props.title}
            project={this.props.project}
          />
        );
      } else {
        return (
          <Timer
            title={this.props.title}
            project={this.props.project}
            elapsed={this.props.elapsed}
            runningSince={this.props.runningSince}
          />
        );
      }
    }
  }
```

##### TimerForm
```javascript
  class TimerForm extends React.Component {
    render() {
      const submitText = this.props.title ? 'Update' : 'Create';
      return (
        <div className='ui centered card'>
          <div className='content'>
            <div className='ui form'>
              <div className='field'>
                <label>Title</label>
                <input type='text' defaultValue={this.props.title} />
              </div>
              <div className='field'>
                <label>Project</label>
                <input type='text' defaultValue={this.props.project} />
              </div>
              <div className='ui two bottom attached buttons'>
                <button className='ui basic blue button'>
                  {submitText}
                </button>
                <button className='ui basic red button'>
                  Cancel
                </button>
              </div>
            </div>
          </div>
        </div>
      );
    }
  }
```

##### ToggleableTimerForm
```javascript
  class ToggleableTimerForm extends React.Component {
    render() {
      if (this.props.isOpen) {
        return (
          <TimerForm />
        );
      } else {
        return (
          <div className='ui basic content center aligned segment'>
            <button className='ui basic button icon'>
              <i className='plus icon' />
            </button>
          </div>
        );
      }
    }
  }
```
- The return statement under the else block is the markup to render a “+” button. 
- You could make a case that this should be its own React component (say PlusButton) but at present we’ll keep the code inside ToggleableTimerForm.

##### Timer
```javascript
  class Timer extends React.Component {
    render() {
      const elapsedString = helpers.renderElapsedString(this.props.elapsed);
      return (
        <div className='ui centered card'>
          <div className='content'>
            <div className='header'>
              {this.props.title}
            </div>
            <div className='meta'>
              {this.props.project}
            </div>
            <div className='center aligned description'>
              <h2>
                {elapsedString}
              </h2>
            </div>
            <div className='extra content'>
              <span className='right floated edit icon'>
                <i className='edit icon' />
              </span>
              <span className='right floated trash icon'>
                <i className='trash icon' />
              </span>
            </div>
          </div>
          <div className='ui bottom attached blue basic button'>
            Start
          </div>
        </div>
      );
    }
  }
```
- We use a function defined in **helpers.js, renderElapsedString()**. 
- The string it renders is in the format ‘HH:MM:SS’.

---

#### #3.1. [STEP 3]: DETERMINE WHAT SHOULD BE STATEFUL

##### TimersDashboard
- In our static app, this declares two child components. 
- It sets **one prop**, which is the **isOpen boolean** that is **passed down to ToggleableTimerForm**.
  
##### EditableTimerList
- This **declares two child components**, each which have props corresponding to a given timer’s properties.

##### EditableTimer
- This uses the prop **editFormOpen**. 

##### Timer
- This uses **all the props for a timer**. 

##### TimerForm
- This has **two interactive input fields**, one for **title** and one for **project**. 
- When editing an existing timer, these fields are initialized with the timer’s current values.

##### STATE CRITERIA
- We can apply criteria to determine if data should be stateful:

**1. Is it passed in from a parent via props? If so, it probably isn’t state.**
- A lot of the data used in our child components are already listed in their parents. 
- This criterion helps us de-duplicate.
- For example, “timer properties” is listed multiple times. 
- When we see the properties declared in EditableTimerList, we can consider it state. But when we see it elsewhere, it’s not.
  
**2. Does it change over time? If not, it probably isn’t state.**
- This is a key criterion of stateful data: it changes.

**3. Can you compute it based on any other state or props in your component? If so, it’s not state.**
- For simplicity, we want to strive to represent state with as few data points as possible.


##### APPLYING THE CRITERIA

##### TimersDashboard
- *isOpen boolean for ToggleableTimerForm*
- **Stateful**. 
- The data is defined here. 
- It changes over time. 
- And it cannot be computed from other state or props.

##### EditableTimerList
- *Timer properties*
- **Stateful**. 
- The data is defined in this component, changes over time, and cannot be computed from other state or props.

##### EditableTimer
- *editFormOpen for a given timer*
- **Stateful**. 
- The data is defined in this component, changes over time, and cannot be computed from other state or props.
  
##### Timer
- *Timer properties*
- In this context, **not stateful**. 
- Properties are passed down from the parent.
  
##### TimerForm
- We might be tempted to conclude that TimerForm doesn’t manage any stateful data, as title and project are props passed down from the parent. 
- However, as we’ll see, forms are special state managers in their own right.
- So, outside of TimerForm, we’ve identified our stateful data:
  - The list of timers and properties of each timer
  - Whether or not the edit form of a timer is open 
  - Whether or not the create form is open

---

#### #4.1. [STEP 4]: DETERMINE IN WHICH COMPONENT EACH PIECE OF STATE SHOULD LIVE

- We can apply the following steps from Facebook’s guide “Thinking in React35” to help us with the process:
- For each piece of state:
  1. Identify every component that renders something based on that state.
  2. Find a common owner component (a single component above all the
components that need the state in the hierarchy).
  3. Either the common owner or another component higher up in the
hierarchy should own the state.
  4. If you can’t find a component where it makes sense to own the state,
create a new component simply for holding the state and add it somewhere in the hierarchy above the common owner component.

##### The list of timers and properties of each timer
- Therefore, **TimersDashboard** is truly the **common owner**. 
- It will render **EditableTimerList** by passing down the timer state. 
- It can handle modifications from **EditableTimerList** and creates from **ToggleableTimerForm**, mutating the state. 
- The new state will flow downward through **EditableTimerList**.

##### Whether or not the edit form of a timer is open
- In our static app, **EditableTimerList** specifies whether or not an EditableTimer should be rendered with its edit form open. 
- Technically, though, this state could just live in each individual EditableTimer. No parent component in the hierarchy depends on this data.
- Storing the state in EditableTimer will be fine for our current needs. But there are a few requirements that might require us to “hoist” this state up higher in the component hierarchy in the future.
- For instance, what if we wanted to impose a restriction such that only one edit form could be open at a time? Then it would make sense for EditableTimerList to own the state, as it would need to inspect it to determine whether to allow a new “edit form open” event to succeed. 
- If we wanted to allow only one form open at all, including the create form, then we’d hoist the state up to TimersDashboard.

##### Visibility of the create form
- TimersDashboard doesn’t appear to care about whether ToggleableTimerForm is open or closed. It feels safe to reason that the state can just live inside ToggleableTimerForm itself.
  
**So, in summary, we’ll have three pieces of state each in three different components:**
- **Timer** data will be owned and managed by **TimersDashboard**.
- Each **EditableTimer** will manage the state of its timer edit form.
- The **ToggleableTimerForm** will manage the state of its form visibility.

---

#### #5.1. [STEP 5]: HAR-CODE INITIAL STATES

##### Adding state to TimerDashboard
```javascript
class TimersDashboard extends React.Component {
  state = {
    timers: [
      {
        title: 'Practice squat',
        project: 'Gym Chores',
        id: uuid.v4(),
        elapsed: 5456099,
        runningSince: Date.now(),
      },
      {
        title: 'Bake squash',
        project: 'Kitchen Chores',
        id: uuid.v4(),
        elapsed: 1273998,
        runningSince: null,
      },
    ],
  };
  render() {
    return (
      <div className="ui three column centered grid">
        {' '}
        <div className="column">
          <EditableTimerList timers={this.state.timers} />
          <ToggleableTimerForm />
        </div>
      </div>
    );
  }
}
```

##### Receiving props in EditableTimerList
```javascript
class EditableTimerList extends React.Component {
  render() {
    const timers = this.props.timers.map((timer) => (
      <EditableTimer
        key={timer.id}
        id={timer.id}
        title={timer.title}
        project={timer.project}
        elapsed={timer.elapsed}
        runningSince={timer.runningSince}
      />
    ));
    return <div id="timers"> {timers}</div>;
  }
}
```

##### Adding state to EditableTimer
```javascript
class EditableTimer extends React.Component {
  state = {
    editFormOpen: false,
  };
  render() {
    if (this.state.editFormOpen) {
      return (
        <TimerForm
          id={this.props.id}
          title={this.props.title}
          project={this.props.project}
        />
      );
    } else {
      return (
        <Timer
          id={this.props.id}
          title={this.props.title}
          project={this.props.project}
          elapsed={this.props.elapsed}
          runningSince={this.props.runningSince}
        />
      );
    }
  }
}
```

##### Timer remains stateless

##### Adding state to ToggableTimerForm
```javascript
class ToggleableTimerForm extends React.Component {
  state = {
    isOpen: false,
  };

  handleFormOpen = () => {
    this.setState({ isOpen: true });
  };

  render() {
    if (this.state.isOpen) {
      return (
        <TimerForm />
      );
    } else {
      return (
        <div className='ui basic content center aligned segment'>
          <button
            className='ui basic button icon'
            onClick={this.handleFormOpen}
          >
            <i className='plus icon' />
          </button>
        </div>
      );
    }
  }
}
```

##### Adding state to TimerForm
```javascript
class TimerForm extends React.Component {
  state = {
    title: this.props.title || '',
    project: this.props.project || '',
  };

  handleTitleChange = (e) => {
    this.setState({ title: e.target.value });
  };

  handleProjectChange = (e) => {
    this.setState({ project: e.target.value });
  };

  render() {
    const submitText = this.props.title ? 'Update' : 'Create';
    return (
      <div className='ui centered card'>
        <div className='content'>
          <div className='ui form'>
            <div className='field'>
              <label>Title</label>
              <input
                type='text'
                value={this.state.title}
                onChange={this.handleTitleChange}
              />
            </div>
            <div className='field'>
              <label>Project</label>
              <input
                type='text'
                value={this.state.project}
                onChange={this.handleProjectChange}
              />
            </div>
            <div className='ui two bottom attached buttons'>
              <button className='ui basic blue button'>
                {submitText}
              </button>
              <button className='ui basic red button'>
                Cancel
              </button>
            </div>
          </div>
        </div>
      </div>
    );
  }
}

```

To recap, here’s an example of the lifecycle of TimerForm:
1. On the page is a timer with the title “Mow the lawn.”
2. The user toggles open the edit form for this timer, mounting TimerForm to the
page.
3. TimerForm initializes the state property title to the string "Mow the lawn".
4. The user modifies the input field, changing it to the value "Cut the grass".
5. With every keystroke, React invokes handleTitleChange. The internal state of
title is kept in-sync with what the user sees on the page.

---

#### #6.1. [STEP 6]: ADD INVERSE DATA FLOW

We are going to need inverse data flow in two areas:
- **TimerForm** needs to propagate create and update events (create while under ToggleableTimerForm and update while under EditableTimer). 
- Both events will eventually reach **TimersDashboard**.
- **Timer** has a fair amount of behavior. It needs to handle delete and edit clicks, as well as the start and stop timer logic.

##### TimerForm
- **TimerForm** needs two event handlers:
  1. When the form is submitted (creating or updating a timer) 
  2. When the “Cancel” button is clicked (closing the form)

- **TimerForm** will receive two functions as props to handle each event. The parent component that uses TimerForm is responsible for providing these functions:
  1. props.onFormSubmit(): called when the form is submitted
  2. props.onFormClose(): called when the “Cancel” button is clicked

##### ToggableTimerForm
- First, we’ll modify ToggleableTimerForm. 
- We need it to pass down two prop-functions to TimerForm, **onFormClose()** and **onFormSubmit()**:

##### ToggableTimerForm
- The first event we’re concerned with is the submission of a form. 
- When this happens, either a new timer is being created or an existing one is being updated. 
- We’ll use two separate functions to handle the two distinct events:
  1. handleCreateFormSubmit() will handle creates and will be the function passed to ToggleableTimerForm
  2. handleEditFormSubmit() will handle updates and will be the function passed to EditableTimerList
- Both functions travel down their respective component hierarchies until they reach TimerForm as the prop onFormSubmit().