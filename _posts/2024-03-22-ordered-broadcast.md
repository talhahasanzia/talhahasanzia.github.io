---
date: 22/03/2024
title: Android -  Understanding Ordered Broadcasts
categories: [Android, APIs]
description: Special broadcasts that can be prioritized, stopped propagation and even modified for next receiver.
tags: ['android', 'broadcasts','orderedbroadcast', 'receivers']
---

###  Understanding Ordered Broadcasts
Broadcasts in Android are powerful tools for inter-component communication, allowing different parts of an application or even different applications to send and receive messages. Among the types of broadcasts, ordered broadcasts stand out as a mechanism for ensuring a specific order of receiver execution and enabling receivers to manipulate the propagation of the broadcast. In this article, we'll delve into the concept of ordered broadcasts, provide examples, and compare them with normal broadcasts.

### Ordered Broadcasts: What Are They?
Ordered broadcasts are a type of broadcast in Android where the order of receiver execution is maintained. When an ordered broadcast is sent, receivers are invoked one at a time based on their priority levels. Each receiver has the option to abort the broadcast or pass it on to the next receiver.

### In Action
Now, let's see how to send an ordered broadcast:
```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Create the intent for the ordered broadcast
        Intent orderedBroadcastIntent = new Intent("com.example.ORDERED_BROADCAST");
        orderedBroadcastIntent.putExtra("message", "Hello from MainActivity!");

        // Send the ordered broadcast
        sendOrderedBroadcast(orderedBroadcastIntent, null);
    }
}

```

And, how we can abort the broadcast, so that it is not delivered to any other receiver anymore:
```
public class MyOrderedBroadcastReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // Receive the broadcast
        String action = intent.getAction();
        if ("com.example.ORDERED_BROADCAST".equals(action)) {
            // Log that the receiver has received the broadcast
            String message = intent.getStringExtra("message");
            System.out.println("Ordered Receiver Received: " + message);

            // Do some processing or action here
            
            // Example of aborting the broadcast
            abortBroadcast(); // This will prevent further receivers from receiving the broadcast
            // Log that the broadcast has been aborted
            System.out.println("Ordered Receiver Aborted Broadcast");
        }
    }
}
```

In this `MainActivity` class, we create an intent for the ordered broadcast with the action `"com.example.ORDERED_BROADCAST"`. We also put an extra message in the intent. Then, we send this ordered broadcast using `sendOrderedBroadcast()` method.

With this setup, when `MainActivity` sends the ordered broadcast, `MyOrderedBroadcastReceiver` will receive it. In the `MyOrderedBroadcastReceiver`, it will log the message received and then abort the broadcast. As a result, no other receivers registered to receive this ordered broadcast will receive it.

Make sure to register `MyOrderedBroadcastReceiver` in the manifest file. Also, ensure appropriate permissions are set if required.
```
<receiver android:name=".MyOrderedBroadcastReceiver">
    <intent-filter>
        <action android:name="com.example.ORDERED_BROADCAST" />
    </intent-filter>
</receiver>
```

#### Modifying broadcast data

Consider example where initial broadcast sender sends a message as `initialMessage` in the following example, receiver overwrites that data so when the next receiver gets this broadcast delivered, it will get updated data.

```
// Sending an Ordered Broadcast with initial data
String initialMessage = "Hello from the sender!";
Intent intent = new Intent("com.example.ORDERED_BROADCAST");
intent.putExtra("message", initialMessage);
sendOrderedBroadcast(intent, null);
```
Receiver manipulating data:
```
public class OrderedBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if ("com.example.ORDERED_BROADCAST".equals(action)) {
            // Retrieve the initial message from the broadcast
            String initialMessage = intent.getStringExtra("message");

            // Log.d initialMessage : Hello from the sender!


            // Modify the message
            String modifiedMessage = initialMessage.toUpperCase() + "-Modified; // Example modification

            // Pass the modified message to the next receiver
            setResultData(modifiedMessage);
        }
    }
}
```
Now if the next receiver receives broadcast and prints the data it will get updated value:
```
public class OrderedBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if ("com.example.ORDERED_BROADCAST".equals(action)) {
            // Retrieve the initial message from the broadcast
            String modifiedMessage = intent.getStringExtra("message");

            // Log.d modifiedMessage : HELLO FROM THE SENDER!-Modified

            // This receiver can also abort further deliveries
            abortBroadcast();
        }
    }
}
```

A Kotlin specific [gist available here](https://gist.github.com/talhahasanzia/cd1d4de42b5141c00039da847653623d).

### When to Use Ordered Broadcasts
Ordered broadcasts are useful when:

- A specific order of receiver execution is required.
- Receivers need to manipulate or filter the broadcast before passing it on.
- It's necessary to ensure that only one receiver handles the broadcast.

### Comparison with normal broadcast
While both types of broadcasts serve the purpose of inter-component communication, ordered broadcasts offer distinct advantages over normal broadcasts, especially when precise control and sequencing are paramount.

Normal broadcasts (`sendBroadcast()`) are delivered to all registered receivers simultaneously and independently, without any predefined order or control over propagation. On the other hand, ordered broadcasts (`sendOrderedBroadcast()`) enable receivers to prioritize their execution and influence the delivery of the broadcast to subsequent receivers through methods like `abortBroadcast()`.

### Conclusion
Ordered broadcasts in Android empower developers with the ability to orchestrate the delivery of messages among components in a controlled and sequenced manner. By leveraging the structured delivery mechanism of ordered broadcasts, developers can ensure precise handling of events and streamline the execution flow within their applications.

In scenarios where sequential execution or manipulation of broadcast propagation is required, ordered broadcasts emerge as a powerful tool in the Android developer's arsenal.

### Further Readings
[Broadcasts Overview](https://developer.android.com/develop/background-work/background-tasks/broadcasts)

[Broadcast Receiver](https://developer.android.com/reference/android/content/BroadcastReceiver)

[Ordered Broadcast](https://developer.android.com/reference/android/content/Context#sendOrderedBroadcast(android.content.Intent,%20java.lang.String,%20java.lang.String,%20android.content.BroadcastReceiver,%20android.os.Handler,%20int,%20java.lang.String,%20android.os.Bundle))

[Modifying Broadcast with setResult](https://developer.android.com/reference/android/content/BroadcastReceiver#setResult(int,%20java.lang.String,%20android.os.Bundle))
