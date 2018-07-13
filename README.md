# deploying-rails-react-to-heroku
Instructions for deploying to Heroku

## Requirements

1. Download Heroku CLI (`brew install heroku/brew/heroku`)
1. Log in with `heroku login`

## Deploying a Rails/React app to Heroku

1. Create Rails project (which will be the root project directory) - `rails new project-name --api -T --database=postgresql` (-T to not include tests, and database must be postgres for Heroku)
2. `cd project-name`
3. Create React frontend - `npx create-react-app`
4. Create root `package.json` - `npm init -y`
5. Add Heroku config to root `package.json` (need to replace existing "scripts"):

```
"engines": {
  "node": "10.5.0"
},
"scripts": {
  "build": "cd client && npm install && npm run build && cd ..",
  "deploy": "cp -a client/build/. public/",
  "postinstall": "npm run build && npm run deploy && echo 'Client built!'"
}
```

6. Add Procfile - `touch Procfile`, then add content to Procfile:

```
web: bundle exec rails s
```

7. Add proxy for react app development - Open `client/package.json`, then add somewhere:

```
"proxy": {
  "/": {
    "target": "http://localhost:3001"
  }
}
```

8. Initialize git repository - `git init`
9. Create Heroku project - `heroku create name-of-project`
10. Add buildpacks (tell Heroku which languages it needs to know about) -

```
heroku buildpacks:add heroku/nodejs --index 1
heroku buildpacks:add heroku/ruby --index 2
```

11. Commit changes:

```
git add .
git commit -m "initial commit"
```

12. Push to Heroku - `git push heroku master`
13. `heroku open`

### Development instructions and testing a request

In root project folder:

1. `rails g model user name`
2. `rails g controller users`
3. `rake db:create`
5. Edit `db/seeds` and add `User.create(name: 'Michael')`
6. `rake db:migrate`
7. `rake db:seed`
8. In `app/controllers/users_controller.rb`, add:

```
def index
  render json: User.all
end
```

9. In 'config/routes.rb', add `resources :users, only: ['index']`

8. `rails s -p 3001`
9. In another terminal, in `client` folder - `yarn start`
10. Edit `client/src/App.js` and add:

```
componentDidMount() {
  fetch("/users")
    .then(r => r.json())
    .then(json => {
      console.log(json);
    });
}
```

11. Look in browser console and make sure the request is being properly made

### Get this database setup working for Heroku

1. Commit changes: `git add .` and `git commit -m "add seed data"`
2. `git push heroku master`
3. `heroku run rake db:migrate`
4. `heroku run rake db:seed`
5. `heroku open` and then check browser console to make sure request is being properly made
