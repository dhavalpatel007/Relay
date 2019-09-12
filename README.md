# Relay

###### Library which uses modified BLoC pattern to listen to multiple updates in single builder widget.

#### Import

```dart
import 'package:relay/relay.dart';
```

### Example

Through simple implementation you can relay update from station
and subscribe via RelayBuilder.

RelayBuilder Widget can listen to more than one updates you can 
provide observers parameter a list of updates.

Like in below example, 

* first relay builder widget observes only on counter.
* second relay builder widget observes on both counter and name.

```dart
import 'dart:async';

import 'package:flutter/material.dart' hide Action;
import 'package:relay/relay.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Provider(
      manager: StoreManager(
        stores: {
          ExampleStore: () => ExampleStore(),
        },
      ),
      child: MaterialApp(
        title: 'Relay Example',
        home: Example(),
      ),
    );
  }
}

class IncrementAction extends Action {
  IncrementAction() : super(null);
}

class NameAction extends Action {
  NameAction(String name) : super(name);
}

class CounterUpdate extends Update {
  CounterUpdate(int counter) : super(counter);
}

class NameUpdate extends Update {
  NameUpdate(String name) : super(name);
}

class MessageUpdate extends Update {
  MessageUpdate(String message) : super(message);
}

class ExampleStore extends Store {
  int counter = 0;

  @override
  Stream<Update> onAction(Action action) async* {
    if (action is IncrementAction) {
      if (counter < 10) {
        counter++;
        yield CounterUpdate(counter);
      } else {
        final snackBarMessage = 'Maximum Limit Reached';
        yield MessageUpdate(snackBarMessage);
      }
    } else if (action is NameAction) {
      yield NameUpdate(action.params);
    }
  }
}

class Example extends StatefulWidget {
  @override
  ExampleState createState() => ExampleState();
}

class ExampleState extends State<Example> with ProviderMixin<ExampleStore> {
  void onUpdate(Update update) {
    if (update is MessageUpdate)
      Scaffold.of(context).showSnackBar(SnackBar(content: Text(update.data)));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        child: Center(
          child: Column(
            children: <Widget>[
              RelayBuilder<ExampleStore>( // Only Listen To Single Update.
                observer: CounterUpdate,
                builder: (context, data) => Text(data),
              ),
              MultiRelayBuilder<ExampleStore>( // Listen To Multiple Updates, data will be Map<Type,dynamic>.
                observers: [NameUpdate, CounterUpdate],
                builder: (context, data) =>
                    Text('${data[NameUpdate]} : ${data[CounterUpdate]}'),
              ),
              Dispatcher<ExampleStore>( // Getting Store For Dispatching The Actions.
                builder: (context, store) => TextField(
                  onChanged: (name) => store.dispatchAction(NameAction(name)),
                  decoration: InputDecoration(
                    labelText: 'Name',
                  ),
                ),
              )
            ],
          ),
        ),
      ),
      floatingActionButton: Dispatcher<ExampleStore>(
        builder: (context, store) => FloatingActionButton(
          onPressed: () => store.dispatchAction(IncrementAction()),
          child: Icon(Icons.add),
        ),
      ),
    );
  }
}
```

* Widget

Extend ProviderWidget and ProviderState then you can access
the store object in deep hierarchies also.