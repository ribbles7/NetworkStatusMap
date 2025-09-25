using System;
using System.Collections.Generic;
using System.Web.Script.Serialization;
using System.Xml;
using System.Net.NetworkInformation;
using System.Globalization;

namespace LiveNetworkMap
{
    public partial class Default : System.Web.UI.Page
    {
        // Simple location lookup for demo purposes
        private string GetLocation(double lat, double lon)
        {
            // You could use a real geolocation API here
            if (lat == 33.4484 && lon == -112.0740) return "Phoenix, AZ";
            if (lat == 39.0438 && lon == -77.4874) return "Ashburn, VA";
            if (lat == 37.3861 && lon == -122.0839) return "Mountain View, CA";
            if (lat == 37.7749 && lon == -122.4194) return "San Francisco, CA";
            if (lat == 37.7924 && lon == -122.3964) return "San Francisco, CA";
            if (lat == 40.7128 && lon == -74.0060) return "New York, NY";
            if (lat == 34.0522 && lon == -118.2437) return "Los Angeles, CA";
            return "Unknown";
        }

        protected void Page_Load(object sender, EventArgs e)
        {
            try
            {
                var serverList = new List<Server>();
                string xmlFilePath = Server.MapPath("~/servers.xml");

                XmlDocument doc = new XmlDocument();
                doc.Load(xmlFilePath);

                foreach (XmlNode node in doc.SelectNodes("/servers/server"))
                {
                    var ip = node["ip"].InnerText;
                    string status = "Unknown";
                    long pingMs = -1;
                    int failureCount = 0;
                    string offlineMessage = "";
                    try
                    {
                        using (var ping = new Ping())
                        {
                            var start = DateTime.UtcNow;
                            var reply = ping.Send(ip, 1000); // 1 second timeout
                            var end = DateTime.UtcNow;
                            pingMs = reply.RoundtripTime;
                            status = (reply.Status == IPStatus.Success) ? "Online" : "Offline";
                            if (status == "Offline")
                            {
                                failureCount = 1;
                                offlineMessage = "Ping failed. Check network/firewall.";
                            }
                        }
                    }
                    catch
                    {
                        status = "Offline";
                        pingMs = -1;
                        failureCount = 1;
                        offlineMessage = "Ping exception. Host unreachable.";
                    }

                    double lat = Convert.ToDouble(node["lat"].InnerText, CultureInfo.InvariantCulture);
                    double lon = Convert.ToDouble(node["lon"].InnerText, CultureInfo.InvariantCulture);

                    serverList.Add(new Server
                    {
                        Name = node["name"].InnerText,
                        Ip = ip,
                        Role = node["role"].InnerText,
                        Lat = lat,
                        Lon = lon,
                        Status = status,
                        PingMs = pingMs,
                        LastChecked = DateTime.UtcNow.ToString("yyyy-MM-dd HH:mm:ss 'UTC'"),
                        Location = GetLocation(lat, lon),
                        FailureCount = failureCount,
                        OfflineMessage = offlineMessage
                    });
                }

                var jsonSerializer = new JavaScriptSerializer();
                string json = jsonSerializer.Serialize(serverList);

                Response.Clear();
                Response.ContentType = "application/json; charset=utf-8";
                Response.Write(json);
                Response.Flush();
                Context.ApplicationInstance.CompleteRequest();
            }
            catch (Exception ex)
            {
                Response.Clear();
                Response.ContentType = "text/plain";
                Response.Write("Error: " + ex.Message);
                Response.Flush();
                Context.ApplicationInstance.CompleteRequest();
            }
        }
    }
}