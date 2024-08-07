# Weather-App

- A simple weather application built with React JS that fetches weather data from a third-party API. Enter a city name in the search bar and press enter to see the current weather, including the city name, country, date, temperature, and weather conditions.

## Features

- Search for the weather by city name.
- Displays the city name, country, date, temperature, and weather conditions.
- Fetches real-time weather data from a third-party API.

# To See Demo:

- Link: https://weatherapphimanshu.netlify.app/

## Technologies Used

- React JS
- useState Hook
- Fetch API (Implemented) / Axios (can implement)
- CSS for styling

## Installation

- Clone the repository:git clone https://github.com/yourusername/weather-app.git
- Navigate to the project directory: cd my-weather-app
- Install the dependencies : npm install
- Start the development server: npm start

- REACT_APP_WEATHER_API_KEY=your_api_key_here (put key name as per your choice in .env file and put the value)

- Build the Project: npm run build



use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Auth;
use Illuminate\Http\Request;
use App\Models\PricingRequestMaster;
use App\Models\PricingRequestVariantMaster;
use App\Models\PricingRequestImageMaster;

public function createPriceRequest(Request $request)
{
    // Validation
    $request->validate([
        'project_id' => 'required|integer',
        'payment_term_id' => 'required|integer',
        'supply_method_id' => 'required|integer',
        'so_remarks' => 'nullable|string',
        'compititors_images' => 'nullable|array',
        'compititors_images.*' => 'file|mimes:jpeg,jpg,png,pdf|max:3000',
        'Competitors_ids' => 'nullable|array',
        'Competitors_ids.*' => 'integer',
        'items' => 'required|array',
        'items.*.request_type' => 'required|string|in:NOP,Volume',
        'items.*.product_id' => 'required|integer',
        'items.*.variant_id' => 'nullable|integer',
        'items.*.quantity' => 'required|integer',
        'items.*.asking_price' => 'required|numeric',
        'items.*.remarks' => 'nullable|string',
    ]);

    try {
        // Begin Transaction
        DB::beginTransaction();

        // Create the Pricing Request Master record
        $pricingRequest = PricingRequestMaster::create([
            'project_id' => $request->project_id,
            'payment_term_id' => $request->payment_term_id,
            'supply_method_id' => $request->supply_method_id,
            'so_remarks' => $request->so_remarks,
            'competitors' => json_encode($request->Competitors_ids), // Assuming Competitors_ids are stored as JSON
            'status' => 1, // Assuming 1 means active
            'created_by' => Auth::id(),
            'updated_by' => Auth::id(),
        ]);

        // Handle Competitor Images
        $compititors_images = [];
        if ($request->hasFile('compititors_images')) {
            foreach ($request->file('compititors_images') as $file) {
                $path = $file->store('competitor_images', 'public'); // Save file to 'public/competitor_images' directory
                PricingRequestImageMaster::create([
                    'model_id' => null, // Set the appropriate model ID if required
                    'request_id' => $pricingRequest->id,
                    'image_type' => 'competitor',
                    'image_url' => $path,
                    'created_by' => Auth::id(),
                    'updated_by' => Auth::id(),
                ]);
                $compititors_images[] = $path; // Collect image paths for response
            }
        }

        // Handle Items
        foreach ($request->items as $item) {
            $variant_id = $item['request_type'] === 'NOP' ? $item['variant_id'] : null;

            $pricingRequestVariant = PricingRequestVariantMaster::create([
                'request_id' => $pricingRequest->id,
                'product_id' => $item['product_id'],
                'variant_id' => $variant_id,
                'quantity' => $item['quantity'],
                'asking_price' => $item['asking_price'],
                'comment' => $item['remarks'],
                'created_by' => Auth::id(),
                'updated_by' => Auth::id(),
            ]);
        }

        // Commit the transaction
        DB::commit();

        // Prepare the response data
        $response = [
            'project_id' => $request->project_id,
            'payment_term_id' => $request->payment_term_id,
            'supply_method_id' => $request->supply_method_id,
            'so_remarks' => $request->so_remarks,
            'compititors_images' => $compititors_images,
            'Competitors_ids' => $request->Competitors_ids,
            'items' => $request->items,
        ];

        return response()->json(['data' => $response], 201);

    } catch (\Exception $e) {
        // Rollback the transaction if there's an error
        DB::rollBack();
        return response()->json(['error' => 'Failed to create pricing request', 'message' => $e->getMessage()], 500);
    }
}
```

### Key Changes:

- **Validation and Request Handling**:
  - The code directly accesses request data using `$request->field_name` instead of `$validated['field_name']`.
  - The validation rules ensure that the data meets the required format before proceeding.

- **Error Handling**:
  - The try-catch block ensures that any errors during the transaction are caught, the transaction is rolled back, and an appropriate error message is returned.

- **File Uploads**:
  - Competitor images are stored only if present, with the appropriate validation and storage logic applied.

This code now adheres to your request, handling potential errors and maintaining data integrity with transactions and rollback in case of issues.