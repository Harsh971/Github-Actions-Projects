

### Step 1: Set Up Your AWS S3 Bucket

1. Create an S3 bucket in the AWS Management Console.
2. Enable static website hosting for the S3 bucket.
3. Note down your S3 bucket name, AWS region, and the endpoint for the static website.

### Step 2: Configure AWS Credentials in GitHub Secrets

1. Create an IAM user in AWS with programmatic access and the necessary permissions to upload to the S3 bucket.
2. Generate access keys for this IAM user.
3. In your GitHub repository, go to `Settings` > `Security` > `Secrets and variables` > `Actions`.
4. Add the following secrets:
   - `AWS_ACCESS_KEY_ID`: Your IAM user's access key ID.
   - `AWS_SECRET_ACCESS_KEY`: Your IAM user's secret access key.
   - `AWS_REGION`: The region where your S3 bucket is located.
   - `S3_BUCKET_NAME`: Your S3 bucket name.

### Step 3: Create a GitHub Actions Workflow

Create a `.github/workflows/deploy.yml` file in your repository with the following content:

```yaml
name: Deploy to S3

on:
  schedule:
    - cron: '0 0 * * *'  # Runs every 24 hours
  push:
    branches:
      - main  # Also deploy on push to main branch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v3
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}

    - name: Sync S3 bucket
      run: aws s3 sync . s3://${{ secrets.S3_BUCKET_NAME }} --delete
      env:
        S3_BUCKET_NAME: ${{ secrets.S3_BUCKET_NAME }}
```

### Explanation of the Workflow

1. **Trigger on Schedule and Push:** The workflow is triggered every 24 hours using the cron schedule (`cron: '0 0 * * *'`) and also on push to the `main` branch.
2. **Checkout Repository:** The `actions/checkout` action checks out your repository.
3. **Configure AWS Credentials:** The `aws-actions/configure-aws-credentials` action configures the AWS CLI with the credentials stored in your repository's secrets.
4. **Sync S3 Bucket:** The `aws s3 sync` command synchronizes the contents of your repository with the S3 bucket, deleting any files in the bucket that are not present in the repository.

### Step 4: Commit and Push Changes

Commit and push your workflow file to GitHub:

```sh
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions workflow to deploy to S3"
git push origin main
```

### Step 5: Verify the Deployment

1. After pushing the changes, the workflow will run immediately.
2. Check the `Actions` tab in your GitHub repository to monitor the progress and ensure the workflow completes successfully.
3. Verify that your S3 bucket contains the updated files and that the static website is correctly hosting your content.

