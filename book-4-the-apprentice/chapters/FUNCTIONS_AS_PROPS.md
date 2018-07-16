# Passing Functionality as Properties

It's time to add one of the CRUD methods to your application. You're going to start with DELETE. Specifically, you're going to start with deleting animals to remove them from the kennel. You are going to add a Delete link to each animal card.

Before you write code, consider the Single Responsibility Principle again. Which component should have the responsibility of deleting an animal?

* The `<Animal>` component itself?
* The `<AnimalList>` component?

 Think about the flow of the application for case.

When you are viewing the entire list of animals, and you click the Delete link, the animal should be deleted and the entire list should be rendered again with the new data. However, if you are viewing a single animal, and you click the Delete button, then the user should be redirected back to the list of animals.

So each component, **`AnimalList`** and **`Animal`**, need the ability to delete, but what happens afterwards is different.

In this situation, you would define that function in a component that can pass the reference to each one. It will be written in a higher order component.

The first thing you need to do is add a link in each animal card for the customer to click on to check out their animal. In the code below, there is a new `<a>` element  that contains the word *Delete*.

> Animal.js

```js
import React from "react"
import { Link } from "react-router-dom"


export default props => {
    return (
        <div className="card" style={{width: `18rem`}}>
            <div className="card-body">
                <h5 className="card-title">
                    {props.animal.name}
                </h5>
                <p className="card-text">{props.animal.breed}</p>
                <a href="#">Delete</a>
            </div>
        </div>
    )
}
```

Time to add behavior to that link so that when it is clicked, the animal is removed from the API. Remember from above that the responsibility of actually removing the animal from the kennel should reside in the **`AnimalList`** component.

Next, define that function in **`AnimalList`**.

```js
export default class AnimalList extends Component {
    ...

    checkOutAnimal = animalId => {
        // Delete the specified animal from the API
        fetch(`http://localhost:5002/animals/${animalId}`, {
            method: "DELETE"
        })
        // When DELETE is finished, retrieve the new list of animals
        .then(() => {
            // Remember you HAVE TO return this fetch to the subsequenet `then()`
            return fetch("http://localhost:5002/animals")
        })
        // Once the new array of animals is retrieved, set the state
        .then(a => a.json())
        .then(animalList => {
            this.setState({
                animals: animalList
            })
        })
    }

    ...
}
```

Later, you will pass this function reference from **`AnimalList`** to the  **`Animal`** component, so it will be receiving that functionality as a property. For now, however, add an `onClick` attribute to the hyperlink that you added in the **`Animal`** component. This ensure that when the customer clicks on the hyperlink (_which is in the child component_) that the function to remove the animal (_which is defined in the parent component_) is invoked.

```html
<a href="#" onClick={props.checkOutAnimal}>Delete</a>
```

Now your **`AnimalList`** component contains the logic for deleting an animal from the API, and updating its state. The **`Animal`** component is invoking it.

There's one thing that's missing. Notice that the `checkOutAnimal` function accepts one argument: `animalId`. In the **`Animal`** component, you specify that the `checkOutAnimal` function should be invoked when the customer clicks on the delete link, but you haven't specified **_which_** animal should be deleted. Using a lamba is how you can do that.

```html
<a href="#" onClick={() => props.checkOutAnimal(props.animal.id)}>Delete</a>
```

## Function Reference as a Property

Now that the function to perform the delete has been written, and the child component specifies that it should be invoked on click, and the correct argument is passed, it's time to refactor the **`AnimalList`** component to pass the function reference to its child.

> AnimalList.js

```js
render() {
    return (
        <React.Fragment>
            {
                this.state.animals.map(animal =>
                    <Animal key={animal.id}
                            animal={animal}
                            checkOutAnimal={this.checkOutAnimal}
                    />
                )
            }
        </React.Fragment>
    )
}
```

So just like a primitive value, such as a string or an integer, you can pass function references from a parent component to a child component. The child component can then specify when that functionality should be invoked, even though it was defined on its parent.

## Practice: Fire Employees

Add the same functionality to the **`EmployeeList`** and **`Employee`** components so that employees can be fired!

## Practice: Customers

Add a **`CustomerList`** and **`Customer`** component so that the kennel company can keep track of the people who bring in their pets. Make sure that each customer can be removed from the system, if needed.

## Challenge: Add a New Animal

Add a simple form - with an input field for the animal's name and one for the breed - to the **`AnimalList`** component to kennel a new animal. When the submit button is clicked, you should POST the new animal to the API, then immediately GET the new list and set state. Just like you did for DELETE.

#### Reference

* [React Forms](https://reactjs.org/docs/forms.html)
* [How to Work with Forms in React](https://www.sitepoint.com/work-with-forms-in-react/)

