This is a [Next.js](https://nextjs.org/) project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `app/page.tsx`. The page auto-updates as you edit the file.

This project uses [`next/font`](https://nextjs.org/docs/basic-features/font-optimization) to automatically optimize and load Inter, a custom Google Font.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js/) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/deployment) for more details.

## Example of DynamoDB Data
{
  "ID": {
    "S": "4"
  },
  "image": {
    "S": "https://julie-website-mongo.s3.amazonaws.com/pictures/art4.jpg"
  },
  "measurements": {
    "S": "18 x 24"
  },
  "price": {
    "N": "300"
  },
  "title": {
    "S": "Fells Point"
  }
}

## 1st Lambda Function

    import json
    import boto3

    def lambda_handler(event, context):
    dynamodb = boto3.client('dynamodb')

    table_name = 'JustArtArtWork'

    try:
        response = dynamodb.scan(TableName=table_name)
        
        items = [
            {
                key: item[key].get("S") or item[key].get("N") or item[key].get("BOOL")
                for key in item
            }
            for item in response['Items']
        ]

        return {
            'statusCode': 200,
            'body': items  
        }
    except Exception as e:
        print("Error:", e)
        return {
            'statusCode': 500,
            'body': json.dumps({"error": str(e)})
        }

## 1st Lambda Function IAM Inline Policy

{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"dynamodb:Scan",
				"dynamodb:Query"
			],
			"Resource": "arn:aws:dynamodb:us-east-1:144835292678:table/JustArtArtWork"
		}
	]
}

## 2nd Lambda Function

    import boto3

    def lambda_handler(event, context):
    table_name = "JustArtArtWork"
    key = event['id']

    dynamodb = boto3.client('dynamodb')

    try:
        response = dynamodb.get_item(
            TableName=table_name,
            Key={
                'ID': {'S': key}
            }
        )
        
        queried_item = {k: v[list(v.keys())[0]] for k, v in response['Item'].items()}
        
        return queried_item
    
    except Exception as e:
        return {
            'statusCode': 500,
            'body': str(e)
        }


## 2nd Lambda Function IAM Inline Policy

{
    "Version": "2012-10-17",
    "Statement": [
{
            "Effect": "Allow",
            "Action": "dynamodb:GetItem",
            	"Resource": "arn:aws:dynamodb:us-east-1:144835292678:table/JustArtArtWork"
        }
    ]
}

