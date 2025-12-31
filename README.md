# AntiSpoofGPS
app/src/main/java/com/antispoofgps/MainActivity.kt
package com.antispoofgps

import android.bluetooth.BluetoothAdapter
import android.bluetooth.BluetoothSocket
import android.content.Context
import android.location.Location
import android.location.LocationManager
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import java.io.BufferedReader
import java.io.InputStreamReader
import java.util.*

class MainActivity : AppCompatActivity() {

    private lateinit var socket: BluetoothSocket
    private val uuid = UUID.fromString("00001101-0000-1000-8000-00805F9B34FB")

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        connectBT()
        startMock()
    }

    private fun connectBT() {
        val adapter = BluetoothAdapter.getDefaultAdapter()
        val device = adapter.bondedDevices.first { it.name == "HC-05" }
        socket = device.createRfcommSocketToServiceRecord(uuid)
        socket.connect()
    }

    private fun startMock() {
        val mgr = getSystemService(Context.LOCATION_SERVICE) as LocationManager
        mgr.addTestProvider("gps", false,false,false,false,true,true,true,0,5)
        mgr.setTestProviderEnabled("gps", true)

        Thread {
            val reader = BufferedReader(InputStreamReader(socket.inputStream))
            while (true) {
                val line = reader.readLine() ?: continue
                if (line.startsWith("\$GPRMC")) {
                    val p = line.split(",")
                    val lat = p[3].toDouble()/100
                    val lon = p[5].toDouble()/100
                    val loc = Location("gps")
                    loc.latitude = lat
                    loc.longitude = lon
                    loc.accuracy = 1f
                    mgr.setTestProviderLocation("gps", loc)
                }
            }
        }.start()
    }
}
