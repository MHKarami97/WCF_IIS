ServiceThrottlingBehavior.MaxConcurrentSessions Property
========================================================

Gets or sets a value that specifies the maximum number of sessions a ServiceHost object can accept at one time.

``` c#
public:
 property int MaxConcurrentSessions { int get(); void set(int value); };
```

The maximum number of sessions a service host accepts. The default is 100 times the processor count.

[](#examples)Examples
---------------------

The following code example shows the use of ServiceThrottlingBehavior from an application configuration file that sets the MaxConcurrentSessions, MaxConcurrentCalls, and MaxConcurrentInstances properties to 1 as an example. Real-world experience determines what the optimal settings are for any particular application.

``` c#
<configuration>
  <appSettings>
    <!-- use appSetting to configure base address provided by host -->
    <add key="baseAddress" value="http://localhost:8080/ServiceMetadata" />
  </appSettings>
  <system.serviceModel>
    <services>
      <service 
        name="Microsoft.WCF.Documentation.SampleService"
        behaviorConfiguration="Throttled" >
        <host>
          <baseAddresses>
            <add baseAddress="http://localhost:8080/SampleService"/>
          </baseAddresses>
        </host>
        <endpoint
          address=""
          binding="wsHttpBinding"
          contract="Microsoft.WCF.Documentation.ISampleService"
         />
        <endpoint
          address="mex"
          binding="mexHttpBinding"
          contract="IMetadataExchange"
         />
      </service>
    </services>
    <behaviors>
      <serviceBehaviors>
        <behavior  name="Throttled">
          <serviceThrottling 
            maxConcurrentCalls="1" 
            maxConcurrentSessions="1" 
            maxConcurrentInstances="1"
          />
          <serviceMetadata 
            httpGetEnabled="true" 
            httpGetUrl=""
          />
        </behavior>
      </serviceBehaviors>
    </behaviors>
  </system.serviceModel>
</configuration>
```

[](#remarks)Remarks
-------------------

The MaxConcurrentSessions property specifies the maximum number of sessions a ServiceHost object can accept. It is important to understand that sessions in this case does not mean only channels that support reliable sessions (for example, System.ServiceModel.NetNamedPipeBinding supports sessions but does not include reliable sessions).

Each listener object can have one pending channel session that does not count against the value of MaxConcurrentSessions until WCF accepts the channel session and begins processing messages on it. This property is most useful in scenarios that make use of sessions.

When this property is set to a value less than the number of client threads, the requests from multiple clients may get queued in the same socket connection. The requests from the client that has not created a session with the service will be blocked till the service closes its session with the other clients if number of open sessions on the service has reached `MaxConcurrentSessions`. The client requests that are not served get timed out and the service closes the session abruptly.

To avoid this situation, run the client threads from different app domains so that the request messages go into different socket connections.

You can also set the values of this attribute by using the <serviceThrottling element in an application configuration file.
