
# Ex.No:1 To create a employee details fields and to display the employee details using Firebase Database in Android Studio.


## AIM:

To create and display the employee details using Firebase Database in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as HelloWorld and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Display the employee details in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:

## Emp.java

```
package com.example.ex_1;

public class Emp {

    public String id;
    public String name;
    public String age;
    public String salary;

    public Emp() {
    }

    public Emp(String id, String name, String age, String salary) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
}
```

## MainActivity.java

```
package com.example.ex_1;

import android.os.Bundle;
import android.widget.*;

import androidx.appcompat.app.AppCompatActivity;

import com.google.firebase.database.*;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    EditText etName, etAge, etSalary;
    Button btnAdd, btnUpdate, btnDelete;
    ListView listView;

    DatabaseReference dbRef;

    ArrayList<String> items = new ArrayList<>();
    ArrayList<String> keys = new ArrayList<>();

    ArrayAdapter<String> adapter;
    String selectedKey = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        etName = findViewById(R.id.etName);
        etAge = findViewById(R.id.etAge);
        etSalary = findViewById(R.id.etSalary);

        btnAdd = findViewById(R.id.btnAdd);
        btnUpdate = findViewById(R.id.btnUpdate);
        btnDelete = findViewById(R.id.btnDelete);

        listView = findViewById(R.id.listView);

        dbRef = FirebaseDatabase.getInstance()
                .getReference("Employees");

        adapter = new ArrayAdapter<>(
                this,
                android.R.layout.simple_list_item_1,
                items
        );

        listView.setAdapter(adapter);

        // Display employees from Firebase
        dbRef.addValueEventListener(new ValueEventListener() {

            @Override
            public void onDataChange(DataSnapshot snapshot) {
                items.clear();
                keys.clear();

                for (DataSnapshot d : snapshot.getChildren()) {
                    Emp e = d.getValue(Emp.class);

                    if (e != null) {
                        items.add(e.name + " | "
                                + e.age + " | " + e.salary);

                        keys.add(d.getKey());
                    }
                }

                adapter.notifyDataSetChanged();
            }

            @Override
            public void onCancelled(DatabaseError error) {
                Toast.makeText(MainActivity.this,
                        "Read failed: " + error.getMessage(),
                        Toast.LENGTH_LONG).show();
            }
        });

        // Add employee
        btnAdd.setOnClickListener(v -> {

            String name = etName.getText().toString().trim();
            String age = etAge.getText().toString().trim();
            String salary = etSalary.getText().toString().trim();

            if (name.isEmpty() || age.isEmpty()
                    || salary.isEmpty()) {
                Toast.makeText(this,
                        "Please fill all fields",
                        Toast.LENGTH_SHORT).show();
                return;
            }

            String key = dbRef.push().getKey();

            if (key == null) {
                Toast.makeText(this,
                        "Unable to generate employee ID",
                        Toast.LENGTH_SHORT).show();
                return;
            }

            Emp e = new Emp(key, name, age, salary);

            btnAdd.setEnabled(false);

            dbRef.child(key).setValue(e)
                    .addOnSuccessListener(aVoid -> {
                        Toast.makeText(this,
                                "Employee saved successfully!",
                                Toast.LENGTH_SHORT).show();

                        btnAdd.setEnabled(true);

                        // Keep fields filled after adding
                    })
                    .addOnFailureListener(error -> {
                        Toast.makeText(this,
                                "Save failed: "
                                        + error.getMessage(),
                                Toast.LENGTH_LONG).show();

                        btnAdd.setEnabled(true);
                    });
        });

        // Update employee
        btnUpdate.setOnClickListener(v -> {

            if (selectedKey == null) {
                Toast.makeText(this,
                        "Select an employee first",
                        Toast.LENGTH_SHORT).show();
                return;
            }

            String name = etName.getText().toString().trim();
            String age = etAge.getText().toString().trim();
            String salary = etSalary.getText().toString().trim();

            if (name.isEmpty() || age.isEmpty()
                    || salary.isEmpty()) {
                Toast.makeText(this,
                        "Please fill all fields",
                        Toast.LENGTH_SHORT).show();
                return;
            }

            Emp e = new Emp(selectedKey, name, age, salary);

            dbRef.child(selectedKey).setValue(e)
                    .addOnSuccessListener(aVoid -> {
                        Toast.makeText(this,
                                "Employee updated successfully!",
                                Toast.LENGTH_SHORT).show();

                        clearFields();
                    })
                    .addOnFailureListener(error ->
                            Toast.makeText(this,
                                    "Update failed: "
                                            + error.getMessage(),
                                    Toast.LENGTH_LONG).show()
                    );
        });

        // Delete employee
        btnDelete.setOnClickListener(v -> {

            if (selectedKey == null) {
                Toast.makeText(this,
                        "Select an employee first",
                        Toast.LENGTH_SHORT).show();
                return;
            }

            dbRef.child(selectedKey).removeValue()
                    .addOnSuccessListener(aVoid -> {
                        Toast.makeText(this,
                                "Employee deleted successfully!",
                                Toast.LENGTH_SHORT).show();

                        clearFields();
                    })
                    .addOnFailureListener(error ->
                            Toast.makeText(this,
                                    "Delete failed: "
                                            + error.getMessage(),
                                    Toast.LENGTH_LONG).show()
                    );
        });

        // Select employee from the list
        listView.setOnItemClickListener(
                (parent, view, position, id) -> {

                    selectedKey = keys.get(position);

                    dbRef.child(selectedKey).get()
                            .addOnSuccessListener(snapshot -> {
                                Emp e = snapshot.getValue(Emp.class);

                                if (e != null) {
                                    etName.setText(e.name);
                                    etAge.setText(e.age);
                                    etSalary.setText(e.salary);
                                }
                            })
                            .addOnFailureListener(error ->
                                    Toast.makeText(this,
                                            "Unable to load employee",
                                            Toast.LENGTH_SHORT).show()
                            );
                });
    }

    // Clear fields after update or delete
    private void clearFields() {
        etName.setText("");
        etAge.setText("");
        etSalary.setText("");
        selectedKey = null;
    }
}
```

## activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/etName"
        android:layout_width="match_parent"
        android:layout_height="55dp"
        android:hint="Name" />

    <EditText
        android:id="@+id/etAge"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Age"
        android:inputType="number" />

    <EditText
        android:id="@+id/etSalary"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Salary"
        android:inputType="number" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <Button
            android:id="@+id/btnAdd"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Add" />

        <Button
            android:id="@+id/btnUpdate"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Update" />

        <Button
            android:id="@+id/btnDelete"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Delete" />

    </LinearLayout>

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

</LinearLayout>
```



## OUTPUT

<img width="700" height="867" alt="Screenshot 2026-11-03 091528" src="https://github.com/user-attachments/assets/0a0b23b3-bf3d-436b-b838-861eaf458d89" />


<img width="1373" height="821" alt="Screenshot 2026-11-03 091436" src="https://github.com/user-attachments/assets/721d2350-748f-4672-92bb-ace4e005de75" />




## RESULT
Thus a Simple Android Application create a firebase database and to display the employee details using Firbase Real Time Database in Android Studio is developed and executed successfully.
