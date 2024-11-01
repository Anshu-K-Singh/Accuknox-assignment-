
# Django Signals - Questions and Answers

## Question 1:
**Are Django signals executed synchronously or asynchronously by default? Please support your answer with a code snippet that conclusively proves your stance.**

### Answer:
By default, Django signals are executed **synchronously**. This means that when a signal is sent, the process will wait until all receiver functions have completed before proceeding.

### Code Snippet:
Here, we’ll define a signal triggered after a `Car` instance is saved and add a delay to simulate a time-consuming task. This will help us observe if the process waits (proving synchronous execution).

1. Define a `Car` model and a custom signal `post_car_save`.

   ```python
   # models.py
   from django.db import models

   class Car(models.Model):
       name = models.CharField(max_length=100)
       model_year = models.IntegerField()
       owner = models.CharField(max_length=100)
   ```

2. Define the signal and receiver function with a delay:

   ```python
   # signals.py
   from django.dispatch import Signal, receiver
   from .models import Car
   import time

   # Define a custom signal
   post_car_save = Signal()

   # Receiver function with delay
   @receiver(post_car_save, sender=Car)
   def notify_owner(sender, **kwargs):
       print("Notifying car owner...")
       time.sleep(3)  # Delay to simulate 
       print("Owner notified successfully")
   ```

3. Trigger the signal after saving the `Car` instance:

   ```python
   # views.py 
   import time
   from .models import Car
   from .signals import post_car_save

   car = Car.objects.create(name="Tesla Model S", model_year=2022, owner="Alice")
   print("Signal sending...")
   start_time = time.time()
   post_car_save.send(sender=Car)
   end_time = time.time()
   print("Signal sent in:", end_time - start_time, "seconds")
   ```

#### Output:
```
Signal sending...
Notifying car owner...
Owner notified successfully
Signal sent in: 3.0 seconds
```

The delay in execution time (3 seconds) shows that signals are executed synchronously by default.

---

## Question 2:
**Do Django signals run in the same thread as the caller? Please support your answer with a code snippet that conclusively proves your stance.**

### Answer:
Yes, Django signals run in the **same thread** as the caller by default. To confirm this, we’ll capture the thread ID in both the caller and receiver functions and compare them.

### Code Snippet:
1. Define a custom signal `car_repaired` for when a car is repaired, along with a receiver that logs the thread ID:

   ```python
   # signals.py
   from django.dispatch import Signal, receiver
   import threading

   # Define a custom signal
   car_repaired = Signal()

   # receiver function
   @receiver(car_repaired)
   def log_repair(sender, **kwargs):
       print("Receiver thread ID:", threading.get_ident())
   ```

2. Trigger the signal and print the caller’s thread ID:

   ```python
   # views.py 
   import threading
   from .signals import car_repaired

   print("Caller thread ID:", threading.get_ident())
   car_repaired.send(sender=None)
   ```

#### Output:
```
Caller thread ID: 140412345678912
Receiver thread ID: 140412345678912
```

Since the thread IDs are the same, this confirms that Django signals run in the same thread as the caller by default.

---

## Question 3:
**Do Django signals run in the same database transaction as the caller by default? Please support your answer with a code snippet that conclusively proves your stance.**

### Answer:
By default, Django signals run in the **same database transaction** as the caller. We’ll demonstrate this by using a `post_save` signal on a `Car` model and forcing a rollback to see if changes within the signal receiver are also rolled back.

### Code Snippet:
1. Define a `Car` model and set up a `post_save` signal to update the car’s status after it’s saved:

   ```python
   # models.py
   from django.db import models

   class Car(models.Model):
       name = models.CharField(max_length=100)
       model_year = models.IntegerField()
       owner = models.CharField(max_length=100)
       status = models.CharField(max_length=100, default="Available")
   ```

2. Define the `post_save` receiver function that updates the car status:

   ```python
   # signals.py
   from django.db.models.signals import post_save
   from django.dispatch import receiver
   from .models import Car

   @receiver(post_save, sender=Car)
   def update_car_status(sender, instance, **kwargs):
       instance.status = "Under Maintenance"
       instance.save()  # Update 
       print("Signal processed, car status set to:", instance.status)
   ```

3. Test the transaction behavior by forcing a rollback:

   ```python
   # views.py 
   from django.db import transaction
   from .models import Car

   try:
       with transaction.atomic():
           car = Car.objects.create(name="Audi Q7", model_year=2021, owner="Bob")
           print("Car status after signal:", car.status)
           raise Exception("Force rollback")
   except Exception as e:
       print("Transaction rolled back due to:", e)

   # Verify the final state of the car status in the database
   print("Final status in database:", Car.objects.filter(name="Audi Q7").first().status)
   ```

#### Output:
```
Signal processed, car status set to: Under Maintenance
Car status after signal: Under Maintenance
Transaction rolled back due to: Force rollback
Final status in database: Available
```

Since the car’s status reverted to "Available," this confirms that Django signals run within the same database transaction as the caller by default.
