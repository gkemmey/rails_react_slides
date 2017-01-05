name: title
class: center, middle, inverse

# how to use react with rails

---

name: background
class: center, middle, inverse

# background

---

name: background_details
class: middle

.left-column[

## sprockets

]

.right-column[

- rails out-of-the-box asset manager
- processes and bundles javascript into an `application.js` and css into an `application.cs`
- globally exposes all the things
- auto-magically knows to look for assets in `app/assets`, `vendor/assets`, or assets bundled with gems 💎
- at one time, was cutting edge

]

---

name: react_rails
class: center, middle, inverse

# cool, let's just use react like we use everything else
.footnote[enter the `react_rails` gem]

---

name: react_rails_details_01
class: middle

- first party, published by react
- globally exposes `React`
- adds some magic to sprockets for processing `.jsx` files
- globally exposes components in `app/assets/javasacripts/components`

---

name: react_rails_details_02
class: middle

we could do this:

```javascript
class MyComponent extends React.Component
  // ...
  render() {
    return (
      <p> stuffs </p>
    )
  }
}
```

---

name: react_rails_details_02
class: middle

but not this:

```javascript
import { DopeLibrary } from 'some_awesome_npm_thing'

class MyComponent extends React.Component
  // ...
}

export default MyComponent;
```

---

name: react_rails_details_03
class: center, middle, inverse

# we were pretending to write javascript

---

name: react_rails_details_04
class: middle

however, `react_ujs` was awesome 👍

```html+ruby
<div>
  <p>normal server rendered html stuffs</p>
</div>

<%= react_component 'MyComponent', props, html_options %>
```

we could punt to react if and only if it made sense, and do normal rails things for everything else

---

name: react_rails_details_05
class: center, middle, inverse

# we went on this way for a while

---

name: react_rails_details_06
class: middle

```javascript
class InjuredPersonForm extends React.Component {
  constructor(props, context) {
    super(props, context);

    this.updateFirstName     = this.updateInjuredPerson.bind(this, 'firstName');
    this.updateLastName      = this.updateInjuredPerson.bind(this, 'lastName');
    this.updatePhoneNumber   = this.updateInjuredPerson.bind(this, 'phoneNumber');
    this.updateEmailAddress  = this.updateInjuredPerson.bind(this, 'emailAddress');

    this.updateStreetAddress = this.updateInjuredPerson.bind(this, 'streetAddress');
    this.updateCity          = this.updateInjuredPerson.bind(this, 'city');
    this.updateState         = this.updateInjuredPerson.bind(this, 'state');
    this.updateZip           = this.updateInjuredPerson.bind(this, 'zip');

    this.updatePersonType    = this.updateInjuredPerson.bind(this, 'personType');
  }

  updateInjuredPerson(name, event) {
    this.context.updateInjuredPersonsAttributes({
      [this.props.index]: {
        [name]: event.target.value
      }
    })
  }
}

InjuredPersonForm.contextTypes = {
  deselectInjuryToEdit:           React.PropTypes.func,
  updateInjuredPersonsAttributes: React.PropTypes.func,
  meta:                           React.PropTypes.object
}
```

---

name: react_rails_details_07
class: middle

```javascript
IncidentReportContainer.childContextTypes = {		
  changeStep:                           React.PropTypes.func,		
  updateIncidentReportState:            React.PropTypes.func,		
  handleSave:                           React.PropTypes.func,		
  selectInjuryToEdit:                   React.PropTypes.func,		
  deselectInjuryToEdit:                 React.PropTypes.func,		
  updateInjuredPersonsAttributes:       React.PropTypes.func,		
  updateInjuredEmployeesAttributes:     React.PropTypes.func,		
  selectRecordsWithoutDestroyAttribute: React.PropTypes.func,		
  meta:                                 React.PropTypes.object		
}
```

---

name: react_rails_details_08
class: middle

a "simple" radio button 😑

```html
<div className="form-group">		
  <label className="form-check-inline">		
    <input className="form-check-input" type="radio"
      name="wereAnyPeopleInjuredRadio"		
      value={ !this.props.wereAnyPeopleInjured ? "on" : "off" }		
      checked={ !this.props.wereAnyPeopleInjured }		
      onChange={ this.answerNo } /> No		
  </label>		

  <label className="form-check-inline">		
    <input className="form-check-input" type="radio"
      name="wereAnyPeopleInjuredRadio"		
      value={ this.props.wereAnyPeopleInjured ? "on" : "off" }		
      checked={ this.props.wereAnyPeopleInjured }		
      onChange={ this.answerYes } /> Yes		
  </label>		
</div>
```

---

name: react_rails_details_09
class: center, middle, inverse

# so...that wasn't working

---

name: we_should_use_redux
class: middle

some of our pain points seemed well-suited to `redux` and this library called `redux-form`, but how do we go about using it?

- we were using sprockets
- no access to npm
- no `import { Library } from '...'`

---

name: maybe_webpack
class: middle

well, maybe we could use `webpack`

- with or without sprockets
- better than browserify? the `broswerfiy-rails` gem that marries it with sprockets
- via the `react_on_rails` gem?
- so many questions...?

---

name: eventual_solution
class: center, middle, inverse

# sprockets + webpack

---

name: why_keep_sprockets
class: middle

why keep sprockets? 🤔

- let's us keep using gems that bundle things we want (i.e. `bootstrap`)
- we don't need to replicate the things sprockets already does, like asset hashing
- rails ❤️ sprockets: it auto-magically works

---

name: so_whats_that_look_like
class: center, middle, inverse

# so what's that look like?

---

name: directory_structure
class: center

![directory_structure](/public/images/directory_structure.png)

---

name: webpack_config

```javascript
let components = allFilesIn(__dirname + "/app/assets/javascripts/process_with_webpack/root_components/").
  map((path) => {
    return {
      test: require.resolve(path),
      loader: `expose-loader?${camelize(filename(path))}`
    }
  });

let config = {
  entry: {
    application: __dirname + "/app/assets/javascripts/application_entry.js"
  },

  output: {
    path: __dirname + "/app/assets/javascripts",
    filename: "[name]_bundle.js"
  },

  module: {
    loaders: components.concat([
      {
        test: /\.(js|jsx)$/,
        include: __dirname + "/app/assets/javascripts/process_with_webpack",
        loader: "babel",
        query: {
          presets: ["es2015", "react", "stage-2"]
        }
      }
    ])
  }
};

module.exports = config;
```

---

name: application_entry
class: middle

```javascript
// react_ujs is going to expect these and our components to be exported globally, so here we go
require('expose?React!react');
require('expose?ReactDOM!react-dom');

require("script!./process_with_webpack/init.js"); // we should change this to not suck so bad

// react_ujs is going to want all of our react components exposed globally, so go ahead and do that
(
  (context) => { context.keys().map(context) }
)(require.context("./process_with_webpack/root_components", true, /\.(js|jsx)$/));
```

---

name: application_manifest
class: middle

```javascript
//= require jquery
//= require jquery_ujs
//= require turbolinks

//= require bootstrap-sprockets

// application code we processed with webpack
//= require application_bundle

// things we're not processing with webpack
//= require user_table

//= require react_ujs
```

---

name: yarn_node_modules_webpack_watch
class: middle

- using `yarn` to install npm modles
- `node_modules` folder in root app directory git-ignored
- `application_bundle.js` and `application_bundle.js.map` are git-ignored
- no messing with hot-reloading: `webpack -d --watch --config webpack.config.js`

---

name: summary
class: middle

- use rails as we always have
    - mostly server-rendered html, punt to react when it makes sense
    - gem-ified assets still easily included
    - sass assets still managed with sprockets
- use react as a first level citizen
    - access to npm libraries
    - properly imported modules

---

name: questions
class: center, middle, inverse

# questions
