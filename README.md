# Workout App

A full stack app for users to create and track their gym workouts.

The app uses Apollo GraphQL for the API layer while maintaining a local data cache via Redux.

---

#### Tech Used

### [Server](https://github.com/msolorio/workout-app)
- Node
- GraphQL / Apollo
- Prisma ORM
- PostgreSQL

### [Client](https://github.com/msolorio/workout-app-client)

- TypeScript
- React
- Redux - Redux toolkit
- GraphQL / Apollo

#### [Expand Image - Right click to open in new tab](https://raw.githubusercontent.com/msolorio/workout_app/main/readme-assets/workout-app-architecture.png)

![Workout app Architecture](./readme-assets/workout-app-architecture.png)

---

## Tech of Note [WIP]

<!-- ### Data Handling Strategy on the Client


Used Apollo GraphQL hooks for persistent data storage while maintaining a local cache via Redux
  - Decreased load on the server by [xxx%]
  - Enabled nearly instantaneous performance for data reads



Allows for implementing optimistic updates as a future feature
- Updating user data and creating new records client-side before receiving a response from the server

### Redux
- Local cache of state
- Improved performance for data fetching
- Can allow for optimistic updates in the future

**Note:** Apollo GraphQL offers caching and one could say Redux was not necessary for this use case. I chose to implement Redux to practice coordinating the two data stores and to allow for optimistic updates as a future feature.

### Apollo GraphQL
GraphQL enabled complete flexibility in requesting data on the client. However, this meant more work server side to resolve the data for the various fields.



- Greatly enjoyed setting up the GraphQL server.
  - Managing the complexity of related models
  - Allowing for complete flexibility on the client



- Interested in using GraphQL more and learning more about the kinds of problems it solves in real-world scenarios -->


<!-- 
Go over GraphQL implementation
- how GraphQL works
- Response has same shape as the request
- Schema is contract between client and server
- Each field corresponds with a resolver - will resolve the data for that field
- Must manually resolve the fields for related models

- On client
  - Error handling for GraphQL requests
    - Implemented a custom error handling mechanism as
    - an opportunity to more deeply understand error handling with GraphQL

-->

### App Organization - Client

<details>
<summary>Learn More</summary>

<br>

Implemented separate layers for data interaction and component UI, mimicking MVC architecture. Container components managed high level coordination of page level tasks. Various model layers oversaw implementation details of working with data.

#### [Expand Image - Right click to open in new tab](https://raw.githubusercontent.com/msolorio/workout_app/main/readme-assets/client-mvc.png)

![MVC architecture on the client](./readme-assets/client-mvc.png)

#### Models - Redux and GraphQL Models
- For abstracting away vendor specific code for Apollo GraphQL and Redux
- Housing client-side error handling for GraphQL queries and mutations
- Uses React Hooks

#### Models - Client Operations Models
- For managing implementation details of communication between GraphQL and Redux
- Presenting high level operations to the controllers
- Uses React Hooks

#### Container Components (Controllers)
- Retrieving data from the URL
- Calling model methods for setting and retrieving data
- Managing local component state
- Handling events
- Handling redirects
- Pulling in UI and passing data

#### Presentation Components (View)
- Presenting data and styled UI

<br>

#### Code Example - Right click to open in new tab

The CreateWorkout container

[See full code](https://github.com/msolorio/workout_app_client/blob/main/src/pages/ShowWorkout/index.tsx)
```typescript
function CreateWorkout(): JSX.Element {
  const createWorkout = model.Workout.useCreateWorkout()

  const stateObj: State = {
    workoutId: null
  }

  const [state, setState] = useState(stateObj)


  const handleCreateWorkout = async (workoutData: WorkoutType) => {
    const createdWorkout: WorkoutType = await createWorkout(workoutData)

    if (createdWorkout.id) {
      setState({ workoutId: createdWorkout.id })
    }
  }

  if (state.workoutId) return <Redirect to={`/workouts/${state.workoutId}`} />

  return (
    <CreateWorkoutUi handleCreateWorkout={handleCreateWorkout} />
  )
}
```

`useCreateWorkout` method creates a workout with Apollo GraphQL, then stores in Redux. To integrate with Apollo hooks, I used hooks to manage model methods. The `useCreateWorkout` hook is called at the component's top level and returns a function that can be invoked in an event handler.

[See full code](https://github.com/msolorio/workout_app_client/blob/main/src/model/resources/Workout/index.ts)

```typescript
...
useCreateWorkout() {
  const createWorkoutGql = gql.Workout.useCreateWorkout()
  const createWorkoutRdx = rdx.Workout.useCreateWorkout()

  async function createWorkout(workoutData: WorkoutType): Promise<WorkoutOrErrorType> {
    const newWorkout = await createWorkoutGql(workoutData)

    if (!newWorkout.error) {
      createWorkoutRdx(newWorkout)
    }

    return newWorkout
  }

  return createWorkout
},
...
```

</details>

---

### App Organization - Server

<details>
  <summary>Learn More</summary>

Decoupled the GraphQL API layer from the data fetching layer allowing for easy repurposing of components. GraphQL could be switched out for a REST API, or the Prisma / Postgres model could be switched out to accomodate a different database.

#### Code Example
Below shows the resolver and model layer for creating a workout.

**Note:** In the model, closure is used to wrap the model method and grant it error handling with `createHandledQuery`.

```js
// Resolver Code
...
  createWorkout: async (parent, args, context) => {
    const modelArgs = {
      ...args,
      userId: context.userId
    }

    const { createdWorkout } = await Workout.createWorkout(modelArgs)

    return createdWorkout
  },
...

// Model Code
...
async function query({
  name,
  description,
  length,
  location,
  exercises,
  userId
}) {

  const newWorkout = await prisma.workout.create({
    data: {
      name: name,
      description: description,
      length: length,
      location: location,
      userId: Number(userId)
    }
  })

  if (exercises) {
    const formattedExercises = exercises.map(ex => {
      ex.workoutId = Number(newWorkout.id);
      return ex;
    })
  
    await prisma.exercise.createMany({
      data: formattedExercises
    })
  }

  return newWorkout;
}

const createWorkout = createHandledQuery(query)

return createWorkout
...
```
</details>

---

### TypeScript
<!-- 
Made me more conscious of how I was coding

TODO: tighten typescript code
 -->

### Docker

### JWT for Authentication

---

## In-Progress
- Converting backend to TypeScript

## TODO Items
This is an ongoing project with critical and non-critical features still to be built.
- Completely convert the backend to TypeScript
- Gaurd against cross-site scripting for all client inputs
- Gaurd against SQL injection for all client inputs
- Improve related prisma queries to increase performance
- Implement optimistic updates for data mutations with Redux


## Future Implementations and Lessons Learned
- Use ES Modules on server to allow importing of TypeScript interfaces
