The `integration_uri` in the `aws_apigatewayv2_integration` resource specifies the **URI of the backend service** that the API Gateway should forward requests to. In the case of a Lambda function integration, the `integration_uri` is the **invoke ARN** of the Lambda function.

### Example of `integration_uri` for a Lambda Function

If you have a Lambda function defined in your Terraform configuration (e.g., `aws_lambda_function.my_lambda_function`), you can reference its **invoke ARN** using the `invoke_arn` attribute.

Here’s how you can fill in the `integration_uri`:

```hcl
resource "aws_apigatewayv2_integration" "apigw_lambda" {
  api_id = aws_apigatewayv2_api.http_api.id

  integration_uri        = aws_lambda_function.my_lambda_function.invoke_arn
  integration_type       = "AWS_PROXY"
  integration_method     = "POST"
  payload_format_version = "2.0"
}
```

---

### Explanation of Fields

1. **`integration_uri`**:
   - This is the ARN of the Lambda function that API Gateway will invoke.
   - For a Lambda function, the `invoke_arn` attribute provides the correct ARN for API Gateway to invoke the function.

2. **`integration_type`**:
   - Set to `"AWS_PROXY"` for Lambda integrations. This means API Gateway will forward the entire request to the Lambda function and return the function's response to the client.

3. **`integration_method`**:
   - Set to `"POST"` for Lambda integrations. This is the HTTP method that API Gateway uses to invoke the Lambda function.

4. **`payload_format_version`**:
   - Set to `"2.0"` for the latest payload format version, which is recommended for Lambda integrations.

---

### Full Example with Lambda Function

Here’s a complete example that ties everything together:

```hcl
# HTTP API
resource "aws_apigatewayv2_api" "http_api" {
  name          = "${local.name_prefix}-topmovies-api"
  protocol_type = "HTTP"
}

# Lambda Function
resource "aws_lambda_function" "my_lambda_function" {
  function_name = "${local.name_prefix}-topmovies-lambda"
  handler       = "index.handler"
  runtime       = "nodejs14.x"
  role          = aws_iam_role.lambda_exec_role.arn

  # Add your Lambda code here (e.g., via a .zip file or inline)
}

# Lambda Integration
resource "aws_apigatewayv2_integration" "apigw_lambda" {
  api_id = aws_apigatewayv2_api.http_api.id

  integration_uri        = aws_lambda_function.my_lambda_function.invoke_arn
  integration_type       = "AWS_PROXY"
  integration_method     = "POST"
  payload_format_version = "2.0"
}

# IAM Role for Lambda
resource "aws_iam_role" "lambda_exec_role" {
  name = "${local.name_prefix}-lambda-exec-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "lambda.amazonaws.com"
        }
      }
    ]
  })
}

# Attach policies to the IAM role
resource "aws_iam_role_policy_attachment" "lambda_exec_policy" {
  role       = aws_iam_role.lambda_exec_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}
```

---

### Key Points
- The `integration_uri` for a Lambda function is the **invoke ARN** of the Lambda function.
- You can reference the `invoke_arn` attribute of the `aws_lambda_function` resource to populate this field.
- The `integration_type` must be set to `"AWS_PROXY"` for Lambda integrations.
- The `integration_method` is typically `"POST"` for Lambda integrations.

Let me know if you need further clarification!