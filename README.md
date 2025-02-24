import React, { useEffect, useState } from "react";
import { View, Text, Button, StyleSheet } from "react-native";
import MapView, { Marker } from "react-native-maps";
import database from "@react-native-firebase/database";
import Geolocation from "@react-native-community/geolocation";
import { NavigationContainer } from "@react-navigation/native";
import { createStackNavigator } from "@react-navigation/stack";

const Stack = createStackNavigator();

// الشاشة الرئيسية (اختيار السائق أو الراكب)
const HomeScreen = ({ navigation }) => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Track My Bus</Text>
      <Button title="أنا سائق" onPress={() => navigation.navigate("Driver")} />
      <Button title="أنا راكب" onPress={() => navigation.navigate("Passenger")} />
    </View>
  );
};

// شاشة السائق (إرسال الموقع إلى Firebase)
const DriverScreen = () => {
  const [location, setLocation] = useState(null);
  const busId = "bus_123"; // معرف الحافلة

  const updateLocation = () => {
    Geolocation.getCurrentPosition(
      (position) => {
        const { latitude, longitude } = position.coords;
        setLocation({ latitude, longitude });

        // إرسال الموقع إلى Firebase
        database().ref(`/buses/${busId}`).set({ latitude, longitude });
      },
      (error) => console.log(error),
      { enableHighAccuracy: true, timeout: 15000, maximumAge: 10000 }
    );
  };

  useEffect(() => {
    const interval = setInterval(updateLocation, 5000); // تحديث كل 5 ثوانٍ
    return () => clearInterval(interval);
  }, []);

  return (
    <View style={styles.container}>
      <Text>موقعك الحالي:</Text>
      {location ? (
        <Text>{`Lat: ${location.latitude}, Lng: ${location.longitude}`}</Text>
      ) : (
        <Text>جارٍ تحديد الموقع...</Text>
      )}
      <Button title="تحديث الموقع" onPress={updateLocation} />
    </View>
  );
};

// شاشة الراكب (عرض الحافلات على الخريطة)
const PassengerScreen = () => {
  const [buses, setBuses] = useState([]);

  useEffect(() => {
    const busesRef = database().ref("/buses");
    busesRef.on("value", (snapshot) => {
      const data = snapshot.val();
      if (data) {
        setBuses(Object.values(data));
      }
    });

    return () => busesRef.off();
  }, []);

  return (
    <View style={styles.container}>
      <MapView style={styles.map} initialRegion={{ latitude: 36.75, longitude: 3.06, latitudeDelta: 0.1, longitudeDelta: 0.1 }}>
        {buses.map((bus, index) => (
          <Marker key={index} coordinate={{ latitude: bus.latitude, longitude: bus.longitude }} title="حافلة" />
        ))}
      </MapView>
    </View>
  );
};

// التطبيق الرئيسي مع التنقل بين الشاشات
export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Driver" component={DriverScreen} />
        <Stack.Screen name="Passenger" component={PassengerScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

// الأنماط (CSS)
const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: "center", alignItems: "center" },
  title: { fontSize: 24, marginBottom: 20 },
  map: { flex: 1, width: "100%" },
});
