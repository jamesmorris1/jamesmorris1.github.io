---
layout: default
title: Series
---

# Article Series

<style>
.series-box {
  border: 1px solid #ddd;
  padding: 15px;
  margin-bottom: 15px;
  border-radius: 5px;
  background-color: #f9f9f9;
}
.series-box h3 {
  margin-top: 0;
  font-size: 1.1em;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.series-box h3:hover {
  color: #159957;
}
.series-description {
  color: #666;
  font-size: 0.9em;
  margin: 10px 0;
}
.series-details {
  display: none;
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid #ddd;
}
.series-details.open {
  display: block;
}
.series-details ul {
  margin: 10px 0;
  padding-left: 20px;
}
.series-details li {
  margin: 8px 0;
}
.toggle-icon {
  font-size: 0.8em;
  color: #159957;
}
.coming-soon {
  color: #999;
  font-style: italic;
  font-size: 0.85em;
}
</style>

<div class="series-box">
  <h3 onclick="toggleSeries('vcf-networking')">
    VCF Multi-Tenant Networking Deep Dive
    <span class="toggle-icon" id="icon-vcf-networking">▼</span>
  </h3>
  <p class="series-description">
    A comprehensive series exploring VMware Cloud Foundation's multi-tenant networking architecture, breaking down how VCF constructs map to NSX-T components.
  </p>
  
  <div class="series-details" id="vcf-networking">
    <h4>Articles in this series:</h4>
    <ul>
      <li>
        <strong>Part 1: Provider Gateway vs NSX-T Tier-0: Understanding the Foundation</strong> <span class="coming-soon">(Coming Soon)</span>
        <ul>
          <li>What is a Provider Gateway in VCF</li>
          <li>How it maps to Tier-0 gateways</li>
          <li>Edge cluster connectivity</li>
          <li>BGP peering configuration</li>
        </ul>
      </li>
      
      <li>
        <strong>Part 2: IP Spaces and IP Blocks: VCF's Address Management</strong> <span class="coming-soon">(Coming Soon)</span>
        <ul>
          <li>What are IP Spaces</li>
          <li>IP Block allocation and reachability</li>
          <li>Provider vs Organization IP allocations</li>
          <li>Best practices for IP planning</li>
        </ul>
      </li>
      
      <li>
        <strong>Part 3: Transit Gateway: The Organization's Routing Hub</strong> <span class="coming-soon">(Coming Soon)</span>
        <ul>
          <li>What the Transit Gateway does</li>
          <li>SNAT configuration and external connectivity</li>
          <li>Private-TGW IP blocks explained</li>
          <li>Connecting multiple VPCs</li>
        </ul>
      </li>
      
      <li>
        <strong>Part 4: VPC and Subnets: Building Tenant Networks</strong> <span class="coming-soon">(Coming Soon)</span>
        <ul>
          <li>Private VPC construct in VCF</li>
          <li>VPC Gateway role</li>
          <li>Subnet carving and allocation</li>
          <li>Isolation and security boundaries</li>
        </ul>
      </li>
      
      <li>
        <strong>Part 5: Provider vs Organization: Network Creation Responsibilities</strong> <span class="coming-soon">(Coming Soon)</span>
        <ul>
          <li>Clear separation of duties</li>
          <li>What providers must configure</li>
          <li>What organizations control</li>
          <li>Complete network flow walkthrough</li>
        </ul>
      </li>
    </ul>
  </div>
</div>

<script>
function toggleSeries(seriesId) {
  var details = document.getElementById(seriesId);
  var icon = document.getElementById('icon-' + seriesId);
  
  if (details.classList.contains('open')) {
    details.classList.remove('open');
    icon.textContent = '▼';
  } else {
    details.classList.add('open');
    icon.textContent = '▲';
  }
}
</script>

---

*New series and articles added regularly. Check back soon!*
