---
id: reactivesearch
title: "ReactiveSearch Quickstart"
layout: tutorial
sectionid: getting-started
permalink: getting-started/reactivesearch.html
next: getting-started/data.html
nextTitle: "Importing Data"
redirect_from:
    - "getting-started/"
    - "getting-started/reactivesearch"
    - "install"
    - "quickstart"
---

### Step 0: Install ReactiveSearch Native

If you already have your project setup you can install [`reactivesearch-native`](https://www.npmjs.com/package/@appbaseio/reactivesearch-native) module using npm. Alternatively, you can start a new project following the next step.

```bash
npm install @appbaseio/reactivesearch-native
```

or

```bash
yarn add @appbaseio/reactivesearch-native
```

### Step 1: Create Boilerplate with CRNA

We will create a search UI based on a *cars dataset* with ReactiveSearch components.

![Image](https://imgur.com/uKVq6EN.png)

**Caption:** Final image of how the app will look.

We can either add ReactiveSearch to an existing app or create a boilerplate app with [Create React Native App (CRNA)](https://github.com/react-community/create-react-native-app). For this quickstart guide, we will use the CRNA with [Expo client](https://expo.io/tools#client).

```bash
create-react-native-app my-awesome-search && cd my-awesome-search
```

Install the `@appbaseio/reactivesearch-native` repo.

```bash
npm install @appbaseio/reactivesearch-native
```

### Step 2: Adding the first component

Lets add our first ReactiveSearch component: [ReactiveBase](/getting-started/reactivebase.html), it is a backend connector where we can configure the Elasticsearch index / authorization setup.

We will demonstrate creating an index using [appbase.io](https://appbase.io) service, although you can use any Elasticsearch backend within ReactiveBase.

![create an appbase.io app](https://i.imgur.com/r6hWKAG.gif)

**Caption:** For the example that we will build, the app is called **car-store** and the associated read-only credentials are **cf7QByt5e:d2d60548-82a9-43cc-8b40-93cbbe75c34c**. You can browse and clone the dataset into your own app from  [here](https://opensource.appbase.io/dejavu/live/#?input_state=XQAAAAJrAAAAAAAAAAA9iIqnY-B2BnTZGEQz6wkFsta-jK5IyCHPDQHd0vFqnW3IIPckWf81EYz6c9_C1aGQkSbGptS4zcGd_lZI2UVGi7gEHVqkGAZzrbpw4o5m3TwqV4NeFg28vpiRpym93H_qNV7y_gPH___dHIAA).

Lets update our `src/App.js` file to add ReactiveBase component.

```js
import React from 'react';
import { StyleSheet, Text, View } from 'react-native';
import { ReactiveBase } from '@appbaseio/reactivebase-native';

export default class App extends React.Component {
  render() {
    return (
      <ReactiveBase
        app="car-store"
        credentials="cf7QByt5e:d2d60548-82a9-43cc-8b40-93cbbe75c34c"
      >
        <View style={styles.container}>
            <Text>Hello ReactiveSearch!</Text>
        </View>
      </ReactiveBase>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
});
```

This is how the app should look after running the `yarn start` command.

![Screenshot](https://imgur.com/yEVncXC.png)

### Step 3: Adding Filter and Result Components

For this app, we will be using [TextField](/components/textfield.html) component for filtering the dataset and [ReactiveList](/components/reactivelist.html) component for showing the search results.

Lets add them within the ReactiveBase component. But before we do that, lets look at the important props for each.

```js
<TextField
	componentId="searchbox"
	dataField="name"
	placeholder="Search for cars"
/>
```

The [**TextField**](/components/textfield.html) component we describe above creates a searchbox UI component that queries on the `name` field in the dataset.

Next, we need a component to show the matching results. [**ReactiveList**](/components/reactivelist.html) does exactly this.

```js
<ReactiveList
	componentId="results"
	dataField="name"
	size={7}
	showResultStats={false}
	pagination={true}
	react={{
		and: "searchbox"
	}}
	onData={(res) => (
		<View style={styles.result}>
		<Image source={{ uri: 'https://bit.do/demoimg' }} style={styles.image} />
		<View style={styles.item}>
			<Text style={styles.title}>{res.name}</Text>
			<Text>{res.brand + " " + "🌟".repeat(res.rating)}</Text>
		</View>
		</View>
	)}
/>
```

The `react` prop here specifies that it should construct a query based on the current selected values of searchbox and ratingsfilter components. Every time the user changes the input value, a new query is fired -- you don't need to write a manual query for any of the UI components here, although you can override it via `customQuery` prop.

ReactiveSearch uses [native-base](https://docs.nativebase.io/docs/GetStarted.html) which uses some fonts which can be included by adding:

```js
async componentWillMount() {
	await Expo.Font.loadAsync({
		Roboto: require('native-base/Fonts/Roboto.ttf'),
		Roboto_medium: require('native-base/Fonts/Roboto_medium.ttf'),
		Ionicons: require('native-base/Fonts/Ionicons.ttf'),
	});
}
```

Now, we will put all the things together to create the final view.

```js
import React from 'react';
import {
  StyleSheet,
  Text,
  View,
  ScrollView,
  Image
} from 'react-native';
import {
  ReactiveBase,
  TextField,
  ReactiveList
} from '@appbaseio/reactivebase-native';

export default class App extends React.Component {
  constructor() {
    super();
    this.state = {
      isReady: false
    }
  }

  async componentWillMount() {
		await Expo.Font.loadAsync({
			Roboto: require('native-base/Fonts/Roboto.ttf'),
			Roboto_medium: require('native-base/Fonts/Roboto_medium.ttf'),
			Ionicons: require('native-base/Fonts/Ionicons.ttf'),
		});

		this.setState({ isReady: true });
  }

  render() {
    if (!this.state.isReady) {
      return (
        <View style={styles.container}>
          <Text>Loading...</Text>
        </View>
      )
    }

    return (
      <ReactiveBase
        app="car-store"
        credentials="cf7QByt5e:d2d60548-82a9-43cc-8b40-93cbbe75c34c"
      >
        <ScrollView>
          <View style={styles.container}>
            <TextField
              componentId="searchbox"
              dataField="name"
              placeholder="Search for cars"
            />
            <ReactiveList
              componentId="results"
              dataField="name"
              size={7}
              showResultStats={false}
              pagination={true}
              react={{
                and: "searchbox"
              }}
              onData={(res) => (
                <View style={styles.result}>
                  <Image source={{ uri: 'https://bit.do/demoimg' }} style={styles.image} />
                  <View style={styles.item}>
                    <Text style={styles.title}>{res.name}</Text>
                    <Text>{res.brand + " " + "🌟".repeat(res.rating)}</Text>
                  </View>
                </View>
              )}
            />
          </View>
        </ScrollView>
      </ReactiveBase>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    padding: 10,
    marginTop: 25
  },
  image: {
    width: 100,
    height: 100
  },
  result: {
    flexDirection: 'row',
    width: '100%',
    margin: 5,
    alignItems: 'center',
  },
  item: {
    flexDirection: 'column',
    paddingLeft: 10
  },
  title: {
    fontWeight: 'bold'
  }
});
```

If you have followed along so far, you should be able to see the final app:  

![Image](https://imgur.com/uKVq6EN.png)

In the next section we explain about **importing data**.

