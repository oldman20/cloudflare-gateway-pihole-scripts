# Cloudflare Gateway Pi-hole Scripts (CGPS)

![Cloudflare Gateway Analytics screenshot](.github/images/gateway_analytics.png)

Cloudflare Gateway allows you to create custom rules to filter HTTP, DNS, and network traffic based on your firewall policies. This is a collection of scripts that can be used to get a similar experience as if you were using Pi-hole, but with Cloudflare Gateway - so no servers to maintain or need to buy a Raspberry Pi!

## About the individual scripts

- `cf_list_delete.js` - Deletes all lists created by CGPS from Cloudflare Gateway. This is useful for subsequent runs.
- `cf_list_create.js` - Takes an input.csv file containing domains and creates lists in Cloudflare Gateway
- `cf_gateway_rule_create.js` - Creates a Cloudflare Gateway rule to block all traffic if it matches the lists created by CGPS.
- `cf_gateway_rule_delete.js` - Deletes the Cloudflare Gateway rule created by CGPS. Useful for subsequent runs.

## Features

- Support for basic hosts files
- Full support for domain lists
- Automatically cleans up filter lists: removes duplicates, invalid domains, comments and more
- Works fully unattended
- Whitelist support, allowing you to prevent false positives and breakage by forcing trusted domains to always be unblocked.
- Optional health check: Sends a ping request ensuring continuous monitoring and alerting for the workflow execution.


## Usage

### Prerequisites

1. Node.js installed on your machine
2. Cloudflare [Zero Trust](https://one.dash.cloudflare.com/) account - the Free plan is enough. Use the Cloudflare [documentation](https://developers.cloudflare.com/cloudflare-one/) for details.
3. Cloudflare email, API key (NOT the API token), and account ID
4. A file containing the domains you want to block - **max 300,000 domains for the free plan** - in the working directory named `input.csv`. Mullvad provides awesome [DNS blocklists](https://github.com/mullvad/dns-blocklists) that work well with this project. A bash script that downloads recommended blocklists, `get_recommended_filters.sh`, is included.
5. Optional: You can whitelist domains by putting them in a file `whitelist.csv`. You can also use the `get_recomended_whitelist.sh` Bash script to get the recommended whitelists.

### Running locally

1. Clone this repository.
2. Run `npm install` to install dependencies.
3. Copy `.env.example` to `.env` and fill in the values.
4. If this is a subsequent run, execute `node cf_gateway_rule_delete.js` and `node cf_list_delete.js` (in order) to delete old data.
5. If you're on Linux and haven't downloaded any filters yourself, use the `get_recommended_filters.sh` script to download recommended filter lists (about 250 000 domains).
6. Run `node cf_list_create.js` to create the lists in Cloudflare Gateway. This will take a while.
7. Run `node cf_gateway_rule_create.js` to create the firewall rule in Cloudflare Gateway.
8. Profit!

### Running in GitHub Actions

These scripts can be run using GitHub Actions so your filters will be automatically updated and pushed to Cloudflare Gateway. This is useful if you are using a frequently updated malware blocklist.

Please note that the GitHub Action downloads the recommended blocklists and whitelist by default. You can change this behavior by editing the file.

1. Create a new empty, private repository. Forking or public repositories are discouraged, but supported - although the script never leaks your API keys and GitHub Actions secrets are automatically redacted from the logs, it's better to be safe than sorry.
2. Create the following GitHub Actions secrets in your repository settings:

- `CLOUDFLARE_API_KEY`: Your Cloudflare API key
- `CLOUDFLARE_ACCOUNT_ID`: Your Cloudflare account ID
- `CLOUDFLARE_ACCOUNT_EMAIL`: Your Cloudflare account email
- `CLOUDFLARE_LIST_ITEM_LIMIT`: The maximum number of blocked domains allowed for your Cloudflare Zero Trust plan. Use 300000 for the free plan or if you're unsure.
- `PING_URL`: /Optional/ The HTTP(S) URL to ping (using curl) after the GitHub Action has successfully updated your filters. Useful for monitoring.


3. Create a new file in the repository named `.github/workflows/main.yml` with the contents of `auto_update_github_action.yml` found in this repository. The default settings will update your filters every week at 3 AM UTC. You can change this by editing the `schedule` property.
4. Enable GitHub Actions in your repository settings.

### DNS setup for Cloudflare Gateway

1. Go to your Cloudflare Zero Trust dashboard, and navigate to Gateway -> DNS Locations.
2. Click on the default location or create one if it doesn't exist.
3. Configure your router or device based on the provided DNS addresses.

Alternatively, you can install the Cloudflare WARP client and log in to Zero Trust. This method proxies your traffic over Cloudflare servers, meaning it works similarly to a commercial VPN.

## Why not...

### Pi-hole or Adguard Home?

- Complex setup to get it working outside your home
- Requires a Raspberry Pi

### NextDNS?

- DNS filtering is disabled after 300,000 queries per month on the free plan

### Cloudflare Gateway?

- Requires a valid credit card
- Limit of 300k domains on the free plan

### a hosts file?

- Potential performance issues, especially on [Windows](https://github.com/StevenBlack/hosts/issues/93)
- No filter updates
- Doesn't work for your mobile device
- No statistics on how many domains you've blocked

## License

MIT License. See `LICENSE` for more information.

## Donations

If you would like to donate to support this project, you can do so via Liberapay - click the Sponsor button or see my GitHub profile for the link.

## Kiểm tra hiệu năng IP:

Dùng tool [GRC](https://www.grc.com/dns/benchmark.htm) trước khi run click chuột phải chọn test DNSSEC nữa.

Còn kiểm tra hiệu năng DoH thì gắn DoH vào trình duyệt Chrome/Edge (đừng xài Firefox, phân giải Firefox chậm hơn nhiều), rồi vào [dnscheck.tools](https://dnscheck.tools) chờ chạy tới cuối bài test sẽ hiện thời gian ở góc trái phía dưới

Tang này test lấy thông tin về dịch vụ (ECS, thuật toán mã hóa, giao thức kết nối...) là chính chứ test tốc độ không đáng tin cậy, bởi nó làm gì có CDN ở mọi quốc gia mà test tốc độ chuẩn chỉ đơn giản là ping một cái IP của một quốc gia nào đấy

Còn test tốc độ thì cứ ping các trang có CDN VN như Tốc Tốc, Akamai, Shopee.. nếu máy chủ ra ở VN là ngon.

## Check server:
[1.1.1.1](https://1.1.1.1/help)

## Chặn hoặc allow một domain cụ thể
[Video Guide](https://streamable.com/5cz7wd)

[Post voz](https://voz.vn/t/huong-dan-dung-cloudflare-zero-trust.822971/post-27071761)

## Add Private DNS to Android TV:

[https://one.dash.cloudflare.com](https://one.dash.cloudflare.com)

[https://my.nextdns.io](https://my.nextdns.io)

Copy full DoT address next to DNS-over-TLS/QUIC

```
adb shell 
settings put global private_dns_mode hostname
```

Then use this:

`settings put global private_dns_specifier FireTV--StickHD-YOUR-ID.dns.nextdns.io`

Or this:

`settings put global private_dns_specifier YOUR_ID.cloudflare-gateway.com`

Disable Private DNS: 

`settings put global private_dns_mode off`

## Test adblock:
[https://d3ward.github.io/toolz/adblock.html](https://d3ward.github.io/toolz/adblock.html)
