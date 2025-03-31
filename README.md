# FitPal
Building a full-fledged FitPal App for both iOS and Android involves several components, including Flutter for cross-platform development, Firebase for cloud storage, TensorFlow Lite for ML models, HealthKit (iOS), and Google Fit (Android) for health data integration.  
flutter create fitpal
cd fitpal
dependencies:
  flutter:
    sdk: flutter
  firebase_core: latest_version
  firebase_auth: latest_version
  cloud_firestore: latest_version
  firebase_messaging: latest_version
  google_sign_in: latest_version
  flutter_local_notifications: latest_version
  health: latest_version
  google_fit: latest_version
  tflite_flutter: latest_version
  provider: latest_version
  animations: latest_version
flutter pub get
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(FitPalApp());
}
<key>NSHealthShareUsageDescription</key>
<string>This app reads your health data.</string>
<key>NSHealthUpdateUsageDescription</key>
<string>This app writes workout data.</string>
import 'package:health/health.dart';

Future<void> fetchHealthData() async {
  HealthFactory health = HealthFactory();
  List<HealthDataType> types = [
    HealthDataType.STEPS,
    HealthDataType.HEART_RATE,
    HealthDataType.CALORIES,
    HealthDataType.SLEEP_ASLEEP,
  ];

  bool access = await health.requestAuthorization(types);
  if (access) {
    List<HealthDataPoint> data = await health.getHealthDataFromTypes(
      DateTime.now().subtract(Duration(days: 7)),
      DateTime.now(),
      types,
    );
    print("Health Data: $data");
  }
}
import 'package:google_fit/google_fit.dart';

void fetchGoogleFitData() async {
  GoogleFit googleFit = GoogleFit();
  bool granted = await googleFit.requestAuthorization();
  if (granted) {
    var steps = await googleFit.getStepCount(DateTime.now());
    print("Steps: $steps");
  }
}
import 'package:tflite_flutter/tflite_flutter.dart';

Future<void> runModel(List<double> inputData) async {
  Interpreter interpreter = await Interpreter.fromAsset('model.tflite');
  var output = List.filled(1, 0.0).reshape([1, 1]);
  interpreter.run(inputData, output);
  print("Predicted Output: ${output[0][0]}");
}
import 'package:cloud_firestore/cloud_firestore.dart';

void saveUserProgress(String userId, int steps, double calories) {
  FirebaseFirestore.instance.collection("users").doc(userId).set({
    "steps": steps,
    "calories": calories,
    "timestamp": Timestamp.now(),
  }, SetOptions(merge: true));
}
import 'package:firebase_messaging/firebase_messaging.dart';

void setupPushNotifications() async {
  FirebaseMessaging messaging = FirebaseMessaging.instance;
  NotificationSettings settings = await messaging.requestPermission();
  
  FirebaseMessaging.onMessage.listen((RemoteMessage message) {
    print("Notification Received: ${message.notification?.title}");
  });
}
import 'package:flutter/material.dart';
import 'package:animations/animations.dart';

class AnimatedScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: PageTransitionSwitcher(
        duration: Duration(milliseconds: 500),
        transitionBuilder: (child, animation, secondaryAnimation) {
          return FadeThroughTransition(
            animation: animation,
            secondaryAnimation: secondaryAnimation,
            child: child,
          );
        },
        child: HomeScreen(),
      ),
    );
  }
}
