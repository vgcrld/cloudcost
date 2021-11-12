# Galileo Cloud Compass Pricing File "Get" Utility

Used to create the `azure.json` and `aws.json` pricing files for the GCC product.

The production destination for these 2 files is in the directory identified by the 
following Galileo.config value:

```ruby
GCC_PRICING_DIR = Galileo.config[:ui]['gcc']['pricing']
```

This location is `<gpe-server-home>/ui/lib/galileo/visualizations/src/gcc/pricing`.

## Installation

Simpley clone the repo to run this standalone:

```bash
git clone git@codebox.galileosuite.com:galileo/gpe-gcc-pricing.git
cd gpe-gcc-pricing
bundle install --path vendor/bundle
mkdir gcc
bundle exec gpe-gcc-pricing

# Format the output (optional)
cd gcc 
jq '.' aws.json > f-aws.json
jq '.' azure.json > f-azure.json

# Move into the gpe-server branch for an MR  (make sure target dir is correct)
mv f-aws.json ~/code/gpe-server/ui/lib/galileo/visualizations/src/gcc/pricing/aws.json
mv f-azure.json ~/code/gpe-server/ui/lib/galileo/visualizations/src/gcc/pricing/azure.json

```

Or you can use the gem directly, add this line to your application's Gemfile.
(this will vary based on where you fetch this from)

```ruby
gem 'gpe-gcc-pricing', :git => "git@codebox.galileosuite.com:galileo/gpe-gcc-pricing.git"
```

And then execute:

    $ bundle install

Or install it yourself as to local:

    $ gem install ~/gpe-gcc-pricing-1.0.0.gem

Or install it yourself to the default gem location:

    $ gem install ~/gpe-gcc-pricing-1.0.0.gem --install-dir <you-local-path-here>

## Usage

Envoke with or without the `-o output_directory` option. The default is the current
location: `./gcc`.

If the `-o <output_directory>` does not exist then you will need to create it before this is run.

```ruby
bundle exec gpe-gcc-pricing [ -o <output_directory> ] 

# or

rm -rf ~/gcc && mkdir ~/gcc && bundle exec gpe-gcc-pricing -o ~/gcc
```

Or to update the files in place for production you can run this as follows (gpe-server-home must be gpe-server root)

```bash
 bundle exec gpe-gcc-pricing -o gpe-server-home/ui/lib/galileo/visualizations/src/gcc/pricing/
```

# Pricing File Formats (AWS and Azure)

The AWS and Azure pricing file formats are similar but not identical. So that the pricing
engine on the gpe-server side has 2 distinct code paths to parse and price each vendor.

The AWS price file caputred here remains intact as is.  It is not modified any way.

The Azure SKU and price data/json is merged together via this (gpe-gcc-pricing) script 
to more closely resemble the way the AWS pricing file is structured. It is JSON
array of product hashes. 

Each product hash in the array has the capabilties or characteristics of a compute or disk
product and attached is a list of product terms.  The systems are searched by the 
characteristics (CPU, Memory) and then the pricing terms are found for each profile in 
the gpe-server pricing functions.

The code to do the overall pricing in gpe-server is very basic. The complexity is the shear number of 
products to search through and JSON to parse. So it's best to understand the Pricing files 
that are outputed by this script (gpe-gcc-pricing) before you try to understand the 
gpe-server pricing process.

Run the script and review a formated version of the `aws.json` and `azure.json` files 
outputed by this code.

## Azure and AWS Authentication

It's important to understand that this script **REQUIRES** that  vendor access and authentication 
are configured and functioning.

### Azure

The authentication for Azure is build into the script. The auth is only permitted to access
the pricing files. It is attached to subscription for `rdavis` on the ATS group Azure / Microsoft
account.

### AWS 

AWS requires that the local user that is running the script have the `$HOME/.aws` configurations
in place. Commands, like the following, should produce valid AWS repsonses from the system you intend
to run `gpe-gcc-pricing`:

```bash
aws ec2 describe-instances
aws s3 ls
```





