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

---

#### #6.2. Updating Timers

##### Adding editability to Timer
- To notify our app that the user wants to edit a timer we need to add an onClick at-
tribute to the span tag of the edit button. 
We anticipate a prop-function, onEditClick():

##### Updating EditableTimer
- Now we’re prepared to update **EditableTimer**. 
- Again, it will display either the **TimerForm** (if we’re editing) or an individual **Timer** (if we’re not editing).
- Let’s add event handlers for both possible child components. 
- For **TimerForm**, we want to handle the form being closed or submitted. 
- For **Timer**, we want to handle the edit icon being pressed:

##### Updating EditableTimerList
- Moving up a level, we make a one-line addition to **EditableTimerList** to send the submit function from **TimersDashboard** to each **EditableTimer**
- **EditableTimerList** doesn’t need to do anything with this event so again we just pass the function on directly.

##### Defining onEditFormSubmit() in TimersDashboard
- Last step with this pipeline is to define and pass down the submit function for edit forms in **TimersDashboard**.
- For **creates**, we have a function that creates a new timer object with the specified attributes and we append this new object to the end of the timers array in the state.
- For **updates**, we need to hunt through the timers array until we find the timer object that is being updated. As mentioned in the last chapter, the state object cannot be updated directly. We have to use setState().
- Therefore, we’ll use **map()** to traverse the array of timer objects. If the timer’s id matches that of the form submitted, we’ll return a new object that contains the timer with the updated attributes. Otherwise we’ll just return the original timer. This new array of timer objects will be passed to setState():
- Remember, it’s important here that we treat state as **immutable**. By creating a new timers object and then using O**bject#assign()** to populate it, we’re not modifying any of the objects sitting in state.

#### #6.3. Deleting Timers
- In **Timer**, we define a function to handle **trash** button click events.
- We’ve yet to define the function that will be set as the prop **onTrashClick()**
- But you can imagine that when this event reaches the top (TimersDashboard), we’re going to need the id to sort out which timer is being deleted. handleTrashClick() provides the id to this function.

##### Implementing the delete function in TimersDashboard
- The last step is to define the function in TimersDashboard that deletes the desired timer from the state array.
- deleteTimer() uses Array’s filter() method to return a new array with the timer object that has an id matching timerId removed:
- Finally, we pass down handleTrashClick() as a prop:

##### Adding timing functionality
- There are several different ways we can implement a timer system. The simplest approach would be to have a function update the elapsed property on each timer every second. But this is severely limited. What happens when the app is closed? The timer should continue “running.”
- This is why we’ve included the timer property runningSince. A timer is initialized with elapsed equal to 0. When a user clicks “Start”, we do not increment elapsed. Instead, we just set runningSince to the start time.
- We can then use the difference between the start time and the current time to render the time for the user. When the user clicks “Stop”, the difference between the start time and the current time is added to elapsed. runningSince is set to null.
- Therefore, at any given time, we can derive how long the timer has been running by taking Date.now() - runningSince and adding it to the total accumulated time (elapsed). We’ll calculate this inside the Timer component.
- For the app to truly feel like a running timer, we want React to constantly perform this operation and re-render the timers. But elapsed and runningSince will not be changing while the timer is running. So the one mechanism we’ve seen so far to trigger a render() call will not be sufficient.
- Instead, we can use React’s forceUpdate() method. This forces a React component to re-render. We can call it on an interval to yield the smooth appearance of a live timer.

##### Adding a forceUpdate() interval to Timer
- helpers.renderElapsedString() accepts an optional second argument, runningSince. It will add the delta of Date.now() - runningSince to elapsed and use the function millisecondsToHuman() to return a string formatted as HH:MM:SS.
We will establish an interval to run forceUpdate() after the component mounts
- In componentDidMount(), we use the JavaScript function setInterval(). This will invoke the function forceUpdate() once every 50 ms, causing the component to re- render. We set the return of setInterval() to this.forceUpdateInterval.
- In componentWillUnmount(), we use clearInterval() to stop the interval this.forceUpdateInte componentWillUnmount() is called before a component is removed from the app. This
will happen if a timer is deleted. We want to ensure we do not continue calling forceUpdate() after the timer has been removed from the page. React will throw
errors.

##### Add start and stop functionality
- The action button at the bottom of each timer should display “Start” if the timer is paused and “Stop” if the timer is running. 
- It should also propagate events when clicked, depending on if the timer is being stopped or started.
- We could build all of this functionality into Timer.

##### Add timer action events to Timer
- Let’s modify Timer, anticipating a new component called TimerActionButton. 
- This button just needs to know if the timer is running. It also needs to be able to propagate two events, onStartClick() and onStopClick(). 
- These events will eventually need to make it all the way up to TimersDashboard, which can modify runningSince on the timer.

##### Create TimerActionButton
- We render one HTML snippet or another based on this.props.timerIsRunning.
- You know the drill. Now we run these events up the component hierarchy, all the way up to TimersDashboard where we’re managing state:

##### Run the events through EditableTimer and EditableTimerList

#### METHODOLOGY REVIEW
1. Break the app into components
We mapped out the component structure of our app by examining the app’s working UI. We then applied the single-responsibility principle to break com- ponents down so that each had minimal viable functionality.
2. Build a static version of the app
Our bottom-level (user-visible) components rendered HTML based on static props, passed down from parents.
3. Determine what should be stateful
We used a series of questions to deduce what data should be stateful. This data
was represented in our static app as props.
4. Determine in which component each piece of state should live
We used another series of questions to determine which component should own
each piece of state. TimersDashboard owned timer state data and ToggleableTimerForm and EditableTimer both held state pertaining to whether or not to render a TimerForm.
5. Hard-code initial states
We then initialized state-owners’ state properties with hard-coded values.
6. Add inverse data flow
We added interactivity by decorating buttons with onClick handlers. These called functions that were passed in as props down the hierarchy from whichever component owned the relevant state being manipulated.
7. The final step is 7. Add server communication. We’ll tackle this in the next chapter.

---