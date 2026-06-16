import 'dart:io';
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'package:device_info_plus/device_info_plus.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // Note: To enable Cloud Firestore and Firebase Auth, uncomment the line below:
  // await Firebase.initializeApp();
  runApp(const FFSensitivityApp());
}

class FFSensitivityApp extends StatelessWidget {
  const FFSensitivityApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'FF Sensitivity Finder',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        brightness: Brightness.dark,
        scaffoldBackgroundColor: const Color(0xFF050505),
        primaryColor: const Color(0xFFFF5C00),
        colorScheme: const ColorScheme.dark(
          primary: Color(0xFFFF5C00),
          secondary: Color(0xFFFF8A00),
          surface: Color(0xFF121212),
          background: Color(0xFF050505),
        ),
        useMaterial3: true,
      ),
      home: const SessionGate(),
    );
  }
}

// Session Gate to toggle login state on startup
class SessionGate extends StatefulWidget {
  const SessionGate({super.key});

  @override
  State<SessionGate> createState() => _SessionGateState();
}

class _SessionGateState extends State<SessionGate> {
  bool _isLoggedIn = false;
  String _userEmail = '';
  String _userName = '';

  @override
  void initState() {
    super.initState();
    _checkSavedSession();
  }

  void _checkSavedSession() async {
    final prefs = await SharedPreferences.getInstance();
    final savedEmail = prefs.getString('logged_in_email');
    final savedName = prefs.getString('logged_in_name');
    if (savedEmail != null && savedName != null) {
      setState(() {
        _isLoggedIn = true;
        _userEmail = savedEmail;
        _userName = savedName;
      });
    }
  }

  void _onLoginSuccess(String email, String name) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString('logged_in_email', email);
    await prefs.setString('logged_in_name', name);
    setState(() {
      _isLoggedIn = true;
      _userEmail = email;
      _userName = name;
    });
  }

  void _onLogout() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.remove('logged_in_email');
    await prefs.remove('logged_in_name');
    setState(() {
      _isLoggedIn = false;
      _userEmail = '';
      _userName = '';
    });
  }

  @override
  Widget build(BuildContext context) {
    if (_isLoggedIn) {
      return MainTabsScreen(
        userEmail: _userEmail,
        userName: _userName,
        onLogout: _onLogout,
      );
    }
    return LoginScreen(onLoginSuccess: _onLoginSuccess);
  }
}

// ---------------- LOGIN SCREEN ----------------
class LoginScreen extends StatefulWidget {
  final Function(String, String) onLoginSuccess;
  const LoginScreen({super.key, required this.onLoginSuccess});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _nameController = TextEditingController();

  void _submitForm() {
    if (_formKey.currentState!.validate()) {
      widget.onLoginSuccess(
        _emailController.text.trim(),
        _nameController.text.trim(),
      );
    } else {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Bhai, dono fields fill karna compulsory hai!'),
          backgroundColor: Color(0xFFFF5C00),
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: SingleChildScrollView(
          padding: const EdgeInsets.symmetric(horizontal: 24.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // Glowing Game Icon
              Container(
                width: 90,
                height: 90,
                decoration: BoxDecoration(
                  shape: BoxShape.circle,
                  color: const Color(0xFFFF5C00).withOpacity(0.15),
                  boxShadow: [
                    BoxShadow(
                      color: const Color(0xFFFF5C00).withOpacity(0.3),
                      blurRadius: 30,
                    )
                  ],
                ),
                child: const Icon(
                  Icons.sports_esports,
                  size: 50,
                  color: Color(0xFFFF5C00),
                ),
              ),
              const SizedBox(height: 16),
              const Text(
                'FF SENSITIVITY',
                style: TextStyle(
                  fontSize: 32,
                  fontWeight: FontWeight.black,
                  letterSpacing: 2.0,
                  color: Color(0xFFFF5C00),
                ),
              ),
              const Text(
                'FINDER PRO',
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                  letterSpacing: 4.0,
                  color: Colors.white,
                ),
              ),
              const SizedBox(height: 8),
              const Text(
                'Calibrate Headshot Game Sensitivities Instantly',
                style: TextStyle(color: Colors.grey, fontSize: 11),
              ),
              const SizedBox(height: 32),

              // Form Card
              Container(
                padding: const EdgeInsets.all(20),
                decoration: BoxDecoration(
                  color: const Color(0xFF121212),
                  borderRadius: BorderRadius.circular(24),
                  border: Border.all(color: const Color(0xFF1C1C24)),
                ),
                child: Form(
                  key: _formKey,
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      const Text(
                        'PILOT REGISTRATION',
                        style: TextStyle(
                          fontSize: 12,
                          fontWeight: FontWeight.black,
                          color: Colors.white,
                          letterSpacing: 1.0,
                        ),
                      ),
                      const SizedBox(height: 4),
                      const Text(
                        '* Bhai har ek chij compulsory hai fill karna.',
                        style: TextStyle(
                          color: Color(0xFFFF5C00),
                          fontSize: 11,
                          fontWeight: FontWeight.w600,
                        ),
                      ),
                      const SizedBox(height: 20),

                      // Email ID Inputs
                      TextFormField(
                        controller: _emailController,
                        keyboardType: TextInputType.emailAddress,
                        style: const TextStyle(color: Colors.white),
                        decoration: _inputDecoration('Gamer Email ID', Icons.email),
                        validator: (v) => v == null || v.isEmpty ? 'Email fill karna compulsory hai' : null,
                      ),
                      const SizedBox(height: 16),

                      // Display Name Inputs
                      TextFormField(
                        controller: _nameController,
                        style: const TextStyle(color: Colors.white),
                        decoration: _inputDecoration('Display Name / Game ID', Icons.person),
                        validator: (v) => v == null || v.isEmpty ? 'Name fill karna compulsory hai' : null,
                      ),
                      const SizedBox(height: 24),

                      // Submit Area Button
                      SizedBox(
                        width: double.infinity,
                        height: 52,
                        child: ElevatedButton(
                          onPressed: _submitForm,
                          style: ElevatedButton.styleFrom(
                            backgroundColor: const Color(0xFFFF5C00),
                            shape: RoundedCornerShape(14),
                          ),
                          child: const Row(
                            mainAxisAlignment: MainAxisAlignment.center,
                            children: [
                              Icon(Icons.check_circle, color: Colors.black),
                              SizedBox(width: 8),
                              Text(
                                'ENTER GAMING HALL',
                                style: TextStyle(
                                  color: Colors.black,
                                  fontWeight: FontWeight.black,
                                  letterSpacing: 0.5,
                                ),
                              ),
                            ],
                          ),
                        ),
                      ),
                      const SizedBox(height: 12),

                      // Google Sign-In Simulation Button
                      SizedBox(
                        width: double.infinity,
                        height: 52,
                        child: OutlinedButton(
                          onPressed: () {
                            widget.onLoginSuccess(
                              'surajicfbcd@gmail.com',
                              'FreeFirePro_Suraj',
                            );
                          },
                          style: OutlinedButton.styleFrom(
                            side: const BorderSide(color: Color(0xFFFF5C00)),
                            shape: RoundedCornerShape(14),
                          ),
                          child: const Text(
                            'SIGN IN WITH GOOGLE',
                            style: TextStyle(
                              color: Color(0xFFFF5C00),
                              fontWeight: FontWeight.black,
                              fontSize: 12,
                            ),
                          ),
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  InputDecoration _inputDecoration(String label, IconData icon) {
    return InputDecoration(
      labelText: label,
      prefixIcon: Icon(icon, color: const Color(0xFFFF5C00)),
      labelStyle: const TextStyle(color: Colors.grey, fontSize: 13),
      focusedBorder: OutlineInputBorder(
        borderSide: const BorderSide(color: Color(0xFFFF5C00)),
        borderRadius: BorderRadius.circular(12),
      ),
      enabledBorder: OutlineInputBorder(
        borderSide: const BorderSide(color: Color(0xFF1C1C24)),
        borderRadius: BorderRadius.circular(12),
      ),
      errorBorder: OutlineInputBorder(
        borderSide: const BorderSide(color: Colors.redAccent),
        borderRadius: BorderRadius.circular(12),
      ),
      focusedErrorBorder: OutlineInputBorder(
        borderSide: const BorderSide(color: Colors.redAccent),
        borderRadius: BorderRadius.circular(12),
      ),
    );
  }
}

// ---------------- MAIN TABS ----------------
class MainTabsScreen extends StatefulWidget {
  final String userEmail;
  final String userName;
  final VoidCallback onLogout;

  const MainTabsScreen({
    super.key,
    required this.userEmail,
    required this.userName,
    required this.onLogout,
  });

  @override
  State<MainTabsScreen> createState() => _MainTabsScreenState();
}

class _MainTabsScreenState extends State<MainTabsScreen> {
  int _currentIndex = 0;

  // Real-time device configuration parameters loaded on initialization
  String brand = 'ASUS';
  String model = 'ROG Phone 6';
  String osVersion = 'Android 13';
  String ram = '12 GB';
  String rom = '256 GB';

  @override
  void initState() {
    super.initState();
    _fetchDeviceSpecs();
  }

  void _fetchDeviceSpecs() async {
    DeviceInfoPlugin deviceInfo = DeviceInfoPlugin();
    try {
      if (Platform.isAndroid) {
        AndroidDeviceInfo androidInfo = await deviceInfo.androidInfo;
        setState(() {
          brand = androidInfo.brand.toUpperCase();
          model = androidInfo.model;
          osVersion = 'Android ${androidInfo.version.release}';
          // Set standardized styling indicators matching high-end hardware
          ram = '8 GB'; 
          rom = '128 GB';
        });
      }
    } catch (_) {}
  }

  @override
  Widget build(BuildContext context) {
    final List<Widget> screens = [
      CalibratorTab(
        brand: brand,
        model: model,
        osVersion: osVersion,
        ram: ram,
        rom: rom,
      ),
      ProfileTab(
        brand: brand,
        model: model,
        osVersion: osVersion,
        ram: ram,
        rom: rom,
        userEmail: widget.userEmail,
        userName: widget.userName,
        onLogout: widget.onLogout,
      ),
    ];

    return Scaffold(
      appBar: AppBar(
        backgroundColor: const Color(0xFF0A0A0A),
        automaticallyImplyLeading: false,
        title: Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                const Text(
                  'FF SENSITIVITY',
                  style: TextStyle(
                    fontSize: 10,
                    fontWeight: FontWeight.bold,
                    color: Color(0xFFFF5C00),
                    letterSpacing: 2.0,
                  ),
                ),
                Text(
                  _currentIndex == 0 ? 'CALIBRATOR PRO' : 'PROFILE SUMMARY',
                  style: const TextStyle(fontSize: 16, fontWeight: FontWeight.black, italic: true),
                ),
              ],
            ),
            Container(
              padding: const EdgeInsets.symmetric(horizontal: 10, py: 4),
              decoration: BoxDecoration(
                color: const Color(0xFF121212),
                borderRadius: BorderRadius.circular(12),
                border: Border.all(color: const Color(0xFFFF5C00).withOpacity(0.2)),
              ),
              child: Text(
                widget.userName.toUpperCase(),
                style: const TextStyle(fontSize: 11, fontWeight: FontWeight.bold),
              ),
            ),
          ],
        ),
      ),
      body: screens[_currentIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) => setState(() => _currentIndex = index),
        backgroundColor: const Color(0xFF121212),
        selectedItemColor: const Color(0xFFFF5C00),
        unselectedItemColor: Colors.grey,
        items: const [
          BottomNavigationBarItem(
            icon: Icon(Icons.flash_on),
            label: 'CALIBRATOR',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.person),
            label: 'SPECS SHEET',
          ),
        ],
      ),
    );
  }
}

// ---------------- TAB 1: CALIBRATOR ----------------
class CalibratorTab extends StatefulWidget {
  final String brand;
  final String model;
  final String osVersion;
  final String ram;
  final String rom;

  const CalibratorTab({
    super.key,
    required this.brand,
    required this.model,
    required this.osVersion,
    required this.ram,
    required this.rom,
  });

  @override
  State<CalibratorTab> createState() => _CalibratorTabState();
}

class _CalibratorTabState extends State<CalibratorTab> {
  String _style = 'All-Rounder';
  String _performance = 'Mid-End';
  bool _isCalibrating = false;
  Map<String, int>? _results;

  final List<String> styles = ['Rush', 'Sniper', 'All-Rounder'];
  final List<String> performances = ['Low-End', 'Mid-End', 'High-End'];

  void _runCalibration() async {
    setState(() {
      _isCalibrating = true;
      _results = null;
    });

    // Experience gaming delay simulation
    await Future.delayed(const Duration(milliseconds: 1500));

    Map<String, int> activeSens;
    if (_style == 'Rush') {
      activeSens = _performance 