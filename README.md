AndroidNfcReaderDemo
====================
This application shows how to read a nfc tag in your application .</br>
There are few new terms that you need to know before you start coding. Few of them are :</br>

NfcAdapter - extending from the object class uses helper getDefaultAdapter(Context) to get the default NFC adapter.</br>
</br></br>
PendingIntent - If you want to use some other application and want to perform some operation then you pass PendingIntent that gives you the right permission and identity.
</br>
IntentFilter -  It is an object create in the xml file mainly in the mainfest file using the intent-filter tag which describes the operations to be performed on the parent component.
</br>
NDEFMessage - NFC Data Exchange Format is a light weight binary format used to encapsulate binary data.
</br>
____________________________________________________________________________________________________________________________________________________________
After this there are few steps that you need to follow:
</br>
Step 1: Give the NFC permission in the manifest file .</br>
Step 2: Create the layout .</br>
Step 3: Create class to initialize and handle the NFC objects .</br>
</br>
____________________________________________________________________________________________________________________________________________________________
Step 1: Give the NFC permission in the manifest file .

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.webonise.nfcreaderdemo"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="10"
        android:targetSdkVersion="17" />

    <uses-feature
        android:name="android.hardware.nfc"
        android:required="true" />

    <uses-permission android:name="android.permission.NFC" />

    <application
       ...
        <activity
           ...
            <intent-filter>
                ...
            </intent-filter>
            <intent-filter>
                <action android:name="android.nfc.action.TAG_DISCOVERED" />
                <action android:name="android.nfc.action.NDEF_DISCOVERED" />
                <action android:name="android.nfc.action.TECH_DISCOVERED" />

                <category android:name="android.intent.category.DEFAULT" />

                <data android:mimeType="text/plain" />
            </intent-filter>
        </activity>
    </application>
____________________________________________________________________________________________________________________________________________________________
Step 2: Create the layout .

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="15dp"
        android:onClick="onClickReadTag"
        android:text="Read tag content" />

____________________________________________________________________________________________________________________________________________________________
Step 3: Create class to initialize and handle the NFC objects .

public class MainActivity extends Activity {
	private NfcAdapter mNfcAdapter;
	private Context context;
	private PendingIntent mNfcPendingIntent;
	private IntentFilter[] mReadTagFilters;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
	...
		mNfcAdapter = NfcAdapter.getDefaultAdapter(this);

		mNfcPendingIntent = PendingIntent.getActivity(this, 0, new Intent(this,
				getClass()).addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP
				| Intent.FLAG_ACTIVITY_CLEAR_TOP), 0);

		IntentFilter ndefDetected = new IntentFilter(
				NfcAdapter.ACTION_NDEF_DISCOVERED);
		ndefDetected.addDataScheme("http");

		IntentFilter techDetected = new IntentFilter(
				NfcAdapter.ACTION_TECH_DISCOVERED);

		mReadTagFilters = new IntentFilter[] { ndefDetected, techDetected };
	}

	public void onClickReadTag(View view) {
		if (mNfcAdapter != null) {
			if (!mNfcAdapter.isEnabled()) {
				new AlertDialog.Builder(this)
						.setTitle("NFC Dialog")
						.setMessage("Sample message")
						.setPositiveButton("Update Settings",
								new DialogInterface.OnClickListener() {
									public void onClick(DialogInterface arg0,
											int arg1) {
										Intent setnfc = new Intent(
												Settings.ACTION_WIRELESS_SETTINGS);
										startActivity(setnfc);
									}
								})
						.setOnCancelListener(
								new DialogInterface.OnCancelListener() {
									public void onCancel(DialogInterface dialog) {
										dialog.dismiss();
									}
								}).create().show();
			}
			mNfcAdapter.enableForegroundDispatch(this, mNfcPendingIntent,
					mReadTagFilters, null);
		} else {
			Toast.makeText(context,
					"Sorry, NFC adapter not available on your device.",
					Toast.LENGTH_SHORT).show();
		}
	}

	@Override
	protected void onNewIntent(Intent intent) {
		super.onNewIntent(intent);
		NdefMessage[] messages = null;
		Parcelable[] rawMsgs = intent
				.getParcelableArrayExtra(NfcAdapter.EXTRA_NDEF_MESSAGES);
		if (rawMsgs != null) {
			messages = new NdefMessage[rawMsgs.length];
			for (int i = 0; i < rawMsgs.length; i++) {
				messages[i] = (NdefMessage) rawMsgs[i];
			}
		}
		if (messages[0] != null) {
			String result = "";
			byte[] payload = messages[0].getRecords()[0].getPayload();

			for (int b = 1; b < payload.length; b++) {
				result += (char) payload[b];
			}
			Toast.makeText(getApplicationContext(), "Tag has : " + result,
					Toast.LENGTH_LONG).show();
		}
	}
}

____________________________________________________________________________________________________________________________________________________________



