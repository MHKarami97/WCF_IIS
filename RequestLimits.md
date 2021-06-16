## Configuration

### Attributes

| Attribute | Description |
| --- | --- |
| `maxAllowedContentLength` | Optional uint attribute. <br><br>Specifies the maximum length of content in a request, in bytes. <br><br>The default value is `30000000`, which is approximately 28.6MB. |
| `maxQueryString` | Optional uint attribute. <br><br>Specifies the maximum length of the query string, in bytes. <br><br>The default value is `2048`. |
| `maxUrl` | Optional uint attribute. <br><br>Specifies maximum length of the URL, in bytes. <br><br>The default value is `4096`. |

### Child Elements

| Element | Description |
| --- | --- |
| `headerlimits` | Optional element.<br><br>Specifies size limits for HTML headers. |

### Configuration Sample

The following example Web.config file will configure IIS to deny access for HTTP requests where the length of the &quot;Content-type&quot; header is greater than 100 bytes.

```
<configuration>
   <system.webServer>
      <security>
         <requestFiltering>
            <requestLimits>
               <headerLimits>
                  <add header="Content-type" sizeLimit="100" />
               </headerLimits>
            </requestLimits>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
```