import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'dart:async';
import 'package:google_fonts/google_fonts.dart';

// Models
class MenuItem {
  String name;
  double price;

  MenuItem({required this.name, required this.price});
}

class OrderItem {
  String name;
  double price;
  int quantity;

  OrderItem({required this.name, required this.price, required this.quantity});
}

class Order {
  List<OrderItem> items;
  DateTime dateTime;

  Order({required this.items, required this.dateTime});

  double get totalPrice {
    return items.fold(0.0, (sum, item) => sum + (item.price * item.quantity));
  }
}

// AppState
class AppState extends ChangeNotifier {
  List<MenuItem> menuItems = [
    MenuItem(name: 'Burger', price: 5.99),
    MenuItem(name: 'Pizza', price: 7.99),
    MenuItem(name: 'Pasta', price: 6.99),
    MenuItem(name: 'Biryani', price: 8.99),
    MenuItem(name: 'Paneer Tikka', price: 7.49),
    MenuItem(name: 'Masala Dosa', price: 5.99),
    MenuItem(name: 'Chole Bhature', price: 6.99),
    MenuItem(name: 'Butter Chicken', price: 9.99),
    MenuItem(name: 'Naan', price: 2.99),
    MenuItem(name: 'Gulab Jamun', price: 3.99),
  ];

  List<Order> orderHistory = [];
  String userName = 'User';

  void addOrder(Order order) {
    orderHistory.add(order);
    notifyListeners();
  }

  void addMenuItem(MenuItem newItem) {
    menuItems.add(newItem);
    notifyListeners();
  }

  void removeMenuItem(MenuItem item) {
    menuItems.remove(item);
    notifyListeners();
  }

  void updateMenuItem(MenuItem updatedItem) {
    int index = menuItems.indexWhere((item) => item.name == updatedItem.name);
    if (index != -1) {
      menuItems[index] = updatedItem;
      notifyListeners();
    }
  }

  void setUserName(String name) {
    userName = name;
    notifyListeners();
  }

  void clearOrderHistory() {
    orderHistory.clear();
    notifyListeners();
  }

  List<MenuItem> get topSellingItems {
    Map<String, int> itemCount = {};

    for (var order in orderHistory) {
      for (var item in order.items) {
        itemCount[item.name] = (itemCount[item.name] ?? 0) + item.quantity;
      }
    }

    List<MenuItem> sortedItems = menuItems.toList();
    sortedItems.sort(
        (a, b) => (itemCount[b.name] ?? 0).compareTo(itemCount[a.name] ?? 0));

    return sortedItems.take(5).toList();
  }

  List<Order> get latestOrders {
    return orderHistory.reversed.take(5).toList();
  }

  double get totalEarnings {
    return orderHistory.fold(0.0, (sum, order) => sum + order.totalPrice);
  }
}

// SplashScreen
class SplashScreen extends StatefulWidget {
  @override
  _SplashScreenState createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    )..forward();

    _animation = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeIn,
    );

    Timer(Duration(seconds: 3), () {
      Navigator.of(context).pushReplacementNamed('/home');
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.orange[600],
      body: Center(
        child: FadeTransition(
          opacity: _animation,
          child: Text(
            'MYB',
            style: GoogleFonts.pacifico(
              textStyle: TextStyle(
                fontSize: 80,
                color: Colors.white,
                fontWeight: FontWeight.bold,
              ),
            ),
          ),
        ),
      ),
    );
  }
}

// NavigationBar
Widget buildNavigationBar(BuildContext context, int selectedIndex) {
  return BottomNavigationBar(
    currentIndex: selectedIndex,
    selectedItemColor: Colors.orange[800],
    unselectedItemColor: Colors.grey,
    backgroundColor: Colors.orange[200],
    onTap: (index) {
      switch (index) {
        case 0:
          Navigator.pushReplacementNamed(context, '/home');
          break;
        case 1:
          Navigator.pushReplacementNamed(context, '/order_history');
          break;
        case 2:
          Navigator.pushReplacementNamed(context, '/new_order');
          break;
        case 3:
          Navigator.pushReplacementNamed(context, '/edit_menu_items');
          break;
        case 4:
          Navigator.pushReplacementNamed(context, '/edit_profile');
          break;
      }
    },
    items: const [
      BottomNavigationBarItem(
        icon: Icon(Icons.home),
        label: 'Home',
      ),
      BottomNavigationBarItem(
        icon: Icon(Icons.history),
        label: 'Order History',
      ),
      BottomNavigationBarItem(
        icon: Icon(Icons.add_shopping_cart),
        label: 'New Order',
      ),
      BottomNavigationBarItem(
        icon: Icon(Icons.edit),
        label: 'Edit Menu',
      ),
      BottomNavigationBarItem(
        icon: Icon(Icons.person),
        label: 'Profile',
      ),
    ],
  );
}

// HomeScreen
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('MYB'),
        backgroundColor: Colors.orange[800],
      ),
      body: Container(
        color: Colors.orange[50],
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Padding(
              padding: const EdgeInsets.all(16.0),
              child: Consumer<AppState>(
                builder: (context, appState, child) {
                  return RichText(
                    text: TextSpan(
                      children: [
                        TextSpan(
                          text: 'Welcome, \n',
                          style: TextStyle(
                            fontSize: 24,
                            color: Colors.orange[800],
                          ),
                        ),
                        TextSpan(
                          text: appState.userName,
                          style: TextStyle(
                            fontSize: 48,
                            color: Colors.orange[900],
                          ),
                        ),
                      ],
                    ),
                  );
                },
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: GestureDetector(
                onTap: () {
                  Navigator.pushReplacementNamed(context, '/new_order');
                },
                child: Card(
                  child: ListTile(
                    title: Text('Top 5 Selling Items'),
                    subtitle: Consumer<AppState>(
                      builder: (context, appState, child) {
                        return Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: appState.topSellingItems.map((item) {
                            return Text(
                                '${item.name} - \₹${item.price.toStringAsFixed(2)}');
                          }).toList(),
                        );
                      },
                    ),
                  ),
                ),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: GestureDetector(
                onTap: () {
                  Navigator.pushReplacementNamed(context, '/order_history');
                },
                child: Card(
                  child: ListTile(
                    title: Text('Latest 5 Order History'),
                    subtitle: Consumer<AppState>(
                      builder: (context, appState, child) {
                        return Column(
                          crossAxisAlignment: CrossAxisAlignment.start,
                          children: appState.latestOrders.map((order) {
                            return Text(
                              'Order on ${order.dateTime} - \₹${order.totalPrice.toStringAsFixed(2)}',
                              style: TextStyle(fontWeight: FontWeight.bold),
                            );
                          }).toList(),
                        );
                      },
                    ),
                  ),
                ),
              ),
            ),
            Spacer(),
            Padding(
              padding: const EdgeInsets.all(16.0),
              child: Consumer<AppState>(
                builder: (context, appState, child) {
                  return Text(
                    'Total Collection: \₹${appState.totalEarnings.toStringAsFixed(2)}',
                    style: TextStyle(
                      fontSize: 24,
                      color: Colors.orange[900],
                      fontWeight: FontWeight.bold,
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
      bottomNavigationBar: buildNavigationBar(context, 0),
    );
  }
}

// EditProfileScreen
class EditProfileScreen extends StatefulWidget {
  @override
  _EditProfileScreenState createState() => _EditProfileScreenState();
}

class _EditProfileScreenState extends State<EditProfileScreen> {
  final _formKey = GlobalKey<FormState>();
  late TextEditingController _nameController;

  @override
  void initState() {
    super.initState();
    final appState = Provider.of<AppState>(context, listen: false);
    _nameController = TextEditingController(text: appState.userName);
  }

  @override
  void dispose() {
    _nameController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Edit Profile'),
        backgroundColor: Colors.orange[800],
      ),
      body: Container(
        color: Colors.orange[50],
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Form(
            key: _formKey,
            child: Column(
              children: [
                TextFormField(
                  controller: _nameController,
                  decoration: InputDecoration(labelText: 'Name'),
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Please enter a name';
                    }
                    return null;
                  },
                ),
                ElevatedButton(
                  onPressed: () {
                    if (_formKey.currentState!.validate()) {
                      Provider.of<AppState>(context, listen: false)
                          .setUserName(_nameController.text);
                      Navigator.pushReplacementNamed(context, '/home');
                    }
                  },
                  child: Text('Save'),
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.orange[800],
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
      bottomNavigationBar: buildNavigationBar(context, 4),
    );
  }
}

// OrderHistoryScreen
class OrderHistoryScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Order History'),
        backgroundColor: Colors.orange[800],
      ),
      body: Container(
        color: Colors.orange[50],
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Consumer<AppState>(
            builder: (context, appState, child) {
              if (appState.orderHistory.isEmpty) {
                return Center(
                  child: Text('No orders yet.'),
                );
              }
              return ListView.builder(
                itemCount: appState.orderHistory.length,
                itemBuilder: (context, index) {
                  final order = appState.orderHistory[index];
                  return Card(
                    child: ListTile(
                      title: Text('Order on ${order.dateTime}'),
                      subtitle: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          ...order.items.map(
                            (item) => Text(
                                '${item.name} - ${item.quantity} x \₹${item.price}'),
                          ),
                          SizedBox(height: 8),
                          Text(
                              'Total: \₹${order.totalPrice.toStringAsFixed(2)}'),
                        ],
                      ),
                    ),
                  );
                },
              );
            },
          ),
        ),
      ),
      bottomNavigationBar: buildNavigationBar(context, 1),
    );
  }
}

// NewOrderScreen
class NewOrderScreen extends StatefulWidget {
  @override
  _NewOrderScreenState createState() => _NewOrderScreenState();
}

class _NewOrderScreenState extends State<NewOrderScreen> {
  List<OrderItem> orderItems = [];

  void addItemToOrder(MenuItem menuItem) {
    setState(() {
      final existingItem = orderItems.firstWhere(
        (item) => item.name == menuItem.name,
        orElse: () => OrderItem(name: '', price: 0, quantity: 0),
      );

      if (existingItem.name.isNotEmpty) {
        existingItem.quantity += 1;
      } else {
        orderItems.add(
            OrderItem(name: menuItem.name, price: menuItem.price, quantity: 1));
      }
    });
  }

  void removeItemFromOrder(OrderItem orderItem) {
    setState(() {
      orderItems.remove(orderItem);
    });
  }

  void placeOrder() {
    final newOrder = Order(items: orderItems, dateTime: DateTime.now());
    Provider.of<AppState>(context, listen: false).addOrder(newOrder);
    Navigator.pushReplacementNamed(context, '/home');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('New Order'),
        backgroundColor: Colors.orange[800],
      ),
      body: Container(
        color: Colors.orange[50],
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            children: [
              Expanded(
                child: Consumer<AppState>(
                  builder: (context, appState, child) {
                    return ListView.builder(
                      itemCount: appState.menuItems.length,
                      itemBuilder: (context, index) {
                        final menuItem = appState.menuItems[index];
                        return Card(
                          child: ListTile(
                            title: Text(
                                '${menuItem.name} - \₹${menuItem.price.toStringAsFixed(2)}'),
                            trailing: IconButton(
                              icon: Icon(Icons.add),
                              onPressed: () => addItemToOrder(menuItem),
                            ),
                          ),
                        );
                      },
                    );
                  },
                ),
              ),
              SizedBox(height: 16),
              Text(
                'Order Summary',
                style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                    color: Colors.orange[900]),
              ),
              Expanded(
                child: ListView.builder(
                  itemCount: orderItems.length,
                  itemBuilder: (context, index) {
                    final orderItem = orderItems[index];
                    return Card(
                      child: ListTile(
                        title: Text(
                            '${orderItem.name} - ${orderItem.quantity} x \₹${orderItem.price.toStringAsFixed(2)}'),
                        trailing: IconButton(
                          icon: Icon(Icons.remove),
                          onPressed: () => removeItemFromOrder(orderItem),
                        ),
                      ),
                    );
                  },
                ),
              ),
              SizedBox(height: 16),
              ElevatedButton(
                onPressed: placeOrder,
                child: Text('Place Order'),
                style: ElevatedButton.styleFrom(
                  backgroundColor: Colors.orange[800],
                ),
              ),
            ],
          ),
        ),
      ),
      bottomNavigationBar: buildNavigationBar(context, 2),
    );
  }
}

// EditMenuItemsScreen
class EditMenuItemsScreen extends StatefulWidget {
  @override
  _EditMenuItemsScreenState createState() => _EditMenuItemsScreenState();
}

class _EditMenuItemsScreenState extends State<EditMenuItemsScreen> {
  final _formKey = GlobalKey<FormState>();
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _priceController = TextEditingController();
  bool _isEditMode = false;
  MenuItem? _currentItem;

  void _addOrUpdateMenuItem() {
    if (_formKey.currentState!.validate()) {
      final name = _nameController.text;
      final price = double.parse(_priceController.text);

      if (_isEditMode && _currentItem != null) {
        final updatedItem = MenuItem(name: name, price: price);
        Provider.of<AppState>(context, listen: false)
            .updateMenuItem(updatedItem);
      } else {
        final newItem = MenuItem(name: name, price: price);
        Provider.of<AppState>(context, listen: false).addMenuItem(newItem);
      }

      _nameController.clear();
      _priceController.clear();
      setState(() {
        _isEditMode = false;
        _currentItem = null;
      });
    }
  }

  void _editMenuItem(MenuItem item) {
    _nameController.text = item.name;
    _priceController.text = item.price.toString();
    setState(() {
      _isEditMode = true;
      _currentItem = item;
    });
  }

  void _deleteMenuItem(MenuItem item) {
    Provider.of<AppState>(context, listen: false).removeMenuItem(item);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Edit Menu Items'),
        backgroundColor: Colors.orange[800],
      ),
      body: Container(
        color: Colors.orange[50],
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            children: [
              Form(
                key: _formKey,
                child: Column(
                  children: [
                    TextFormField(
                      controller: _nameController,
                      decoration: InputDecoration(labelText: 'Name'),
                      validator: (value) {
                        if (value == null || value.isEmpty) {
                          return 'Please enter a name';
                        }
                        return null;
                      },
                    ),
                    TextFormField(
                      controller: _priceController,
                      decoration: InputDecoration(labelText: 'Price'),
                      keyboardType:
                          TextInputType.numberWithOptions(decimal: true),
                      validator: (value) {
                        if (value == null || value.isEmpty) {
                          return 'Please enter a price';
                        }
                        if (double.tryParse(value) == null) {
                          return 'Please enter a valid number';
                        }
                        return null;
                      },
                    ),
                    SizedBox(height: 16),
                    ElevatedButton(
                      onPressed: _addOrUpdateMenuItem,
                      child: Text(_isEditMode ? 'Update Item' : 'Add Item'),
                      style: ElevatedButton.styleFrom(
                        backgroundColor: Colors.orange[800],
                      ),
                    ),
                  ],
                ),
              ),
              SizedBox(height: 16),
              Expanded(
                child: Consumer<AppState>(
                  builder: (context, appState, child) {
                    return ListView.builder(
                      itemCount: appState.menuItems.length,
                      itemBuilder: (context, index) {
                        final item = appState.menuItems[index];
                        return Card(
                          child: ListTile(
                            title: Text(
                                '${item.name} - \₹${item.price.toStringAsFixed(2)}'),
                            trailing: Row(
                              mainAxisSize: MainAxisSize.min,
                              children: [
                                IconButton(
                                  icon: Icon(Icons.edit),
                                  onPressed: () => _editMenuItem(item),
                                ),
                                IconButton(
                                  icon: Icon(Icons.delete),
                                  onPressed: () => _deleteMenuItem(item),
                                ),
                              ],
                            ),
                          ),
                        );
                      },
                    );
                  },
                ),
              ),
            ],
          ),
        ),
      ),
      bottomNavigationBar: buildNavigationBar(context, 3),
    );
  }
}

// Main function
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => AppState(),
      child: MyApp(),
    ),
  );
}

// MyApp
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'MYB',
      theme: ThemeData(
        primarySwatch: Colors.orange,
        textTheme: GoogleFonts.latoTextTheme(
          Theme.of(context).textTheme,
        ),
      ),
      home: SplashScreen(),
      routes: {
        '/home': (context) => HomeScreen(),
        '/edit_profile': (context) => EditProfileScreen(),
        '/order_history': (context) => OrderHistoryScreen(),
        '/new_order': (context) => NewOrderScreen(),
        '/edit_menu_items': (context) => EditMenuItemsScreen(),
      },
    );
  }
}
