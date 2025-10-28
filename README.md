# Core-Java-Mastery
Core Java Mastery Description
import java.util.Arrays;
import java.util.List;
import java.util.Optional;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.lang.reflect.Method;
import java.lang.reflect.Field;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

/**
 * ==============================================================================
 * 1. Custom Exception
 * ==============================================================================
 * A custom exception to be thrown when a data processing error occurs.
 */
class DataProcessingException extends Exception {
    public DataProcessingException(String message) {
        super(message);
    }
}

/**
 * ==============================================================================
 * 2. Enum with Custom Behavior
 * ==============================================================================
 * Defines the type of operation for the DataProcessor and includes a description.
 */
enum OperationType {
    SQUARE_SUM("Calculates the sum of squares of all numbers."),
    AVERAGE("Calculates the arithmetic mean of all numbers."),
    MAX_VALUE("Finds the maximum value in the list.");

    private final String description;

    OperationType(String description) {
        this.description = description;
    }

    public String getDescription() {
        return description;
    }
}

/**
 * ==============================================================================
 * 3. Generic Class (Pair)
 * ==============================================================================
 * A simple generic container for holding two values of potentially different types.
 * Demonstrates type safety and generics.
 */
class Pair<T, U> {
    private final T first;
    private final U second;

    public Pair(T first, U second) {
        this.first = first;
        this.second = second;
    }

    public T getFirst() {
        return first;
    }

    public U getSecond() {
        return second;
    }

    @Override
    public String toString() {
        return "(" + first + ", " + second + ")";
    }
}

/**
 * ==============================================================================
 * 4. Data Processor Utility
 * ==============================================================================
 * Uses Streams, Lambdas, Optional, and the custom Enum/Exception.
 */
class DataProcessor {

    /**
     * Processes a list of Integers based on the specified operation.
     * Demonstrates: Streams, Lambdas (map, reduce), Optional, Custom Exception.
     *
     * @param numbers The list of integers to process.
     * @param type The operation to perform (SQUARE_SUM, AVERAGE, MAX_VALUE).
     * @return An Optional containing the result, or Optional.empty() if the list is empty.
     * @throws DataProcessingException if the list is null.
     */
    public Optional<Double> processData(List<Integer> numbers, OperationType type) throws DataProcessingException {
        if (numbers == null) {
            throw new DataProcessingException("Input list cannot be null for data processing.");
        }
        if (numbers.isEmpty()) {
            return Optional.empty(); // Use Optional to handle the empty list case
        }

        System.out.println("-> Executing: " + type.getDescription());

        switch (type) {
            case SQUARE_SUM:
                // Stream: map(lambda) -> reduce(method reference)
                double sumOfSquares = numbers.stream()
                        .mapToDouble(n -> n * n)
                        .sum();
                return Optional.of(sumOfSquares);

            case AVERAGE:
                // Stream: mapToInt -> average (returns OptionalDouble)
                Optional<Double> average = numbers.stream()
                        .mapToDouble(Integer::doubleValue)
                        .average()
                        .stream() // Convert OptionalDouble to Stream<Double>
                        .findFirst();
                return average;

            case MAX_VALUE:
                // Stream: max(Comparator.naturalOrder)
                Optional<Integer> max = numbers.stream()
                        .max(Integer::compareTo); // Or Comparator.naturalOrder()
                // Convert Optional<Integer> to Optional<Double>
                return max.map(Integer::doubleValue);

            default:
                // Should not happen, but good practice
                return Optional.empty();
        }
    }
}

/**
 * ==============================================================================
 * 5. Concurrency Utility (Simulated Task Runner)
 * ==============================================================================
 * Demonstrates the use of ExecutorService for managing a pool of tasks.
 */
class TaskRunner {

    /**
     * Executes a number of tasks using a fixed-size thread pool.
     * Demonstrates: ExecutorService, Runnable (Lambda), Concurrency handling.
     *
     * @param taskCount The number of tasks to execute.
     */
    public void executeTasks(int taskCount) {
        System.out.println("\n--- 5. Concurrent Task Runner (" + taskCount + " tasks) ---");
        // Create a fixed-size thread pool
        ExecutorService executor = Executors.newFixedThreadPool(4);

        for (int i = 0; i < taskCount; i++) {
            final int taskId = i;
            // Submit a task defined by a lambda expression (Runnable)
            executor.submit(() -> {
                try {
                    String threadName = Thread.currentThread().getName();
                    System.out.println("  [Task " + taskId + "] started by " + threadName);
                    // Simulate work delay
                    Thread.sleep((long) (Math.random() * 100));
                    System.out.println("  [Task " + taskId + "] finished by " + threadName);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    System.err.println("Task " + taskId + " was interrupted.");
                }
            });
        }

        // Shut down the executor and wait for tasks to complete
        executor.shutdown();
        try {
            // Wait for all tasks to finish execution for a maximum of 5 seconds
            if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
                System.err.println("One or more tasks did not finish within the timeout.");
            }
        } catch (InterruptedException e) {
            System.err.println("Interrupted while waiting for tasks to finish.");
            executor.shutdownNow();
        }
    }
}

/**
 * ==============================================================================
 * 6. Reflection Utility (Object Inspector)
 * ==============================================================================
 * Demonstrates basic reflection to inspect an object's structure at runtime.
 */
class ObjectInspector {

    /** Custom Annotation for methods/fields we want to highlight via reflection */
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Reflectable {
        String description() default "No description provided.";
    }

    // Example class to inspect
    public static class ExampleObject {
        @Reflectable(description = "The unique identifier of the object.")
        private final long id = System.currentTimeMillis();

        private String name = "ReflectMe";

        @Reflectable(description = "A core public method.")
        public void executeLogic() {
            System.out.println("  [Method] Logic executed successfully.");
        }

        public String getName() {
            return name;
        }

        private void privateHelper() {
            // Internal use only
        }
    }

    /**
     * Inspects a given object and prints its fields and methods.
     * Demonstrates: Class loading, Field and Method retrieval, Annotations.
     *
     * @param obj The object instance to inspect.
     */
    public void inspect(Object obj) {
        System.out.println("\n--- 6. Reflection Utility (Object Inspector) ---");
        Class<?> clazz = obj.getClass();
        System.out.println("Inspecting Class: " + clazz.getName());

        System.out.println("\n  --- Fields ---");
        // Get all declared fields (public, private, protected)
        for (Field field : clazz.getDeclaredFields()) {
            System.out.print("  > " + field.getType().getSimpleName() + " " + field.getName());

            // Check for the custom annotation
            if (field.isAnnotationPresent(Reflectable.class)) {
                Reflectable annot = field.getAnnotation(Reflectable.class);
                System.out.print(" [ANNOTATED: " + annot.description() + "]");
            }
            System.out.println();
        }

        System.out.println("\n  --- Methods ---");
        // Get all public methods (including those inherited from Object)
        for (Method method : clazz.getMethods()) {
            // Only show methods defined in this class (or annotated)
            if (method.getDeclaringClass().equals(clazz) || method.isAnnotationPresent(Reflectable.class)) {
                System.out.print("  > " + method.getReturnType().getSimpleName() + " " + method.getName() + "()");

                if (method.isAnnotationPresent(Reflectable.class)) {
                    Reflectable annot = method.getAnnotation(Reflectable.class);
                    System.out.print(" [ANNOTATED: " + annot.description() + "]");
                }
                System.out.println();
            }
        }
    }
}


/**
 * ==============================================================================
 * MAIN CLASS: JavaUtilitySuite
 * ==============================================================================
 * Drives the demonstration of all utility components.
 */
public class JavaUtilitySuite {

    public static void main(String[] args) {
        // Sample data for processing
        List<Integer> data = Arrays.asList(10, 20, 30, 40, 50);

        // --- 1. & 4. Data Processing Utility (Streams, Lambdas, Optional, Exception) ---
        DataProcessor processor = new DataProcessor();
        System.out.println("--- 1. & 4. Data Processor Utility ---");

        // A. Handle potential exception (null input)
        try {
            System.out.println("\n(A) Testing Exception Handling (Null Input):");
            processor.processData(null, OperationType.AVERAGE);
        } catch (DataProcessingException e) {
            System.err.println("  Caught expected error: " + e.getMessage());
        }

        // B. Test AVERAGE operation
        try {
            Optional<Double> avgResult = processor.processData(data, OperationType.AVERAGE);
            avgResult.ifPresentOrElse(
                avg -> System.out.printf("  Average Result: %.2f\n", avg),
                () -> System.out.println("  Could not calculate average (list was empty).")
            );
        } catch (DataProcessingException e) {
            System.err.println("Processing Error: " + e.getMessage());
        }

        // C. Test SQUARE_SUM operation
        try {
            Optional<Double> sumResult = processor.processData(data, OperationType.SQUARE_SUM);
            // Optional.map is used to transform the result if present
            String formattedSum = sumResult.map(sum -> String.format("%.0f", sum)).orElse("N/A");
            System.out.println("  Sum of Squares Result: " + formattedSum);
        } catch (DataProcessingException e) {
             System.err.println("Processing Error: " + e.getMessage());
        }

        // D. Test Optional with empty list
        try {
            List<Integer> emptyList = List.of();
            Optional<Double> emptyResult = processor.processData(emptyList, OperationType.MAX_VALUE);
            System.out.println("\n(D) Testing Optional with Empty List:");
            emptyResult.ifPresentOrElse(
                max -> System.out.println("  Max Value: " + max),
                () -> System.out.println("  Result is Optional.empty() for empty list.")
            );
        } catch (DataProcessingException e) {
             System.err.println("Processing Error: " + e.getMessage());
        }


        // --- 3. Generic Class Demonstration ---
        System.out.println("\n--- 3. Generic Class (Pair) Demonstration ---");
        // Generic type inference at work
        Pair<String, Integer> person = new Pair<>("Alice", 30);
        Pair<Double, List<String>> stats = new Pair<>(98.5, Arrays.asList("Math", "Science"));

        System.out.println("Person Pair (String, Integer): " + person);
        System.out.println("Stats Pair (Double, List<String>): " + stats);
        System.out.println("Retrieved First Element (Type-safe): " + person.getFirst().toUpperCase());


        // --- 5. Concurrency Utility Demonstration ---
        TaskRunner runner = new TaskRunner();
        // The output order will be non-deterministic due to concurrency
        runner.executeTasks(10);


        // --- 6. Reflection Utility Demonstration ---
        ObjectInspector inspector = new ObjectInspector();
        ObjectInspector.ExampleObject example = new ObjectInspector.ExampleObject();
        inspector.inspect(example);
        example.executeLogic(); // Also test the method
    }
}
