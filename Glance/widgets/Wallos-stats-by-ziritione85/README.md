![](preview.png)


```yaml
# Wallos Integration for Glance
# This widget displays information from your Wallos subscription manager
# 
# Configuration Instructions:
# 1. Replace {$your-wallos-api-endpoint} with your Wallos server IP or hostname
# 2. Replace {$your-api-key} with your Wallos API key
# 3. Optionally adjust the cache time (default: 5m)
# 4. You can change the currency symbol from € to your preferred currency

- type: custom-api
  title: Wallos
  cache: 5m
  # skip-json-validation: true  # Uncomment if the API returns non-standard JSON
  url: http://${WALLOS_ENDPOINT}/api/subscriptions/get_subscriptions.php?state=0&api_key=${WALLOS_API_KEY}
  template: |
    {{ if eq .Response.StatusCode 200 }}
      <div style="display: flex; justify-content: space-between;">
        <!-- Section Active Subscriptions -->
        <div style="flex: 1; text-align: center;">
          <p>ACTIVE</p>
          <p class="color-positive">
            {{ len (.JSON.Array "subscriptions") }}
          </p>
        </div>

        <!-- Monthly Cost Calculation -->
        <div style="flex: 1; text-align: center;">
          <p>MONTHLY</p>
          <p class="color-positive">
            {{ $total := 0.0 }}
            {{ range .JSON.Array "subscriptions" }}
              {{ $price := .Float "price" }}
              {{ $cycle := .Int "cycle" }}
              {{ $frequency := .Float "frequency" }}
              
              {{ if eq $cycle 1 }}  <!-- Daily -->
                {{ $total = add $total (mul (div $price $frequency) 30.4375) }}
              {{ else if eq $cycle 2 }}  <!-- Weekly -->
                {{ $total = add $total (mul (div $price (mul $frequency 7)) 4.34524) }}
              {{ else if eq $cycle 3 }}  <!-- Monthly -->
                {{ $total = add $total (div $price $frequency) }}
              {{ else if eq $cycle 4 }}  <!-- Yearly -->
                {{ $total = add $total (div $price (mul $frequency 12)) }}
              {{ end }}
            {{ end }}
            {{ printf "%.2f" $total }}€
          </p>
        </div>

        <!-- Annual Cost Calculation -->
        <div style="flex: 1; text-align: center;">
          <p>ANNUAL</p>
          <p class="color-positive">
            {{ printf "%.2f" (mul $total 12) }}€
          </p>
        </div>
      </div>

      <!-- Next Payment -->
      <div style="display: flex; justify-content: space-between; margin-top: 10px;">
        {{ $nextDate := "" }}
        {{ $nextName := "" }}
        {{ $nextPrice := 0.0 }}
        {{ range .JSON.Array "subscriptions" }}
          {{ $currentDate := .String "next_payment" }}
          {{ if or (eq $nextDate "") (lt $currentDate $nextDate) }}
            {{ $nextDate = $currentDate }}
            {{ $nextName = .String "name" }}
            {{ $nextPrice = .Float "price" }}
          {{ end }}
        {{ end }}

        <!-- Next Payment Details -->
        <div style="flex: 1; text-align: center;">
          <p>NEXT</p>
          <p>
            {{ with $nextName }}
              {{ if gt (len .) 15 }}
                {{ slice . 0 15 }}...
              {{ else }}
                {{ . }}
              {{ end }}
            {{ else }}
              N/A
            {{ end }}
          </p>
        </div>
        <div style="flex: 1; text-align: center;">
          <p>DATE</p>
          <p>
            {{ with $parsedDate := $nextDate | parseTime "2006-01-02" }}
              {{ $parsedDate.Format "02/01/2006" }}
            {{ else }}
              {{ $nextDate }}
            {{ end }}
          </p>
        </div>

        <div style="flex: 1; text-align: center;">
          <p>PRICE</p>
          <p>{{ printf "%.2f" $nextPrice }}€</p>
        </div>
      </div>

      <!-- Dynamic Link -->
      <div style="text-align: center; margin-top: 10px;">
        <a href="${WALLOS_URL}" style="text-decoration: none; color: var(--color-primary);">
          Manage Subscriptions →
        </a>
      </div>
    {{ else }}
      <div style="text-align: center;" class="color-negative">
        Error: {{ .Response.StatusCode }} - {{ .Response.Status }}
      </div>
    {{ end }}

```

## Environment variables

- `your-wallos-api-endpoint` - CHANGE
- `your-api-key` - Configure and unique api key in your user profile
