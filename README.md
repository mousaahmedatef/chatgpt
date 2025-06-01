        public static string GetTemplateWithoutTable(string title ,string? url)
        {
            var html = @$"
<!doctype html>
<html>
<head>
	<meta http-equiv=""content-type"" content=""text/html; charset=utf-8"">
  	<meta name=""viewport"" content=""width=device-width, initial-scale=1.0;"">
 	<meta name=""format-detection"" content=""telephone=no""/>

	<style>
/* Reset styles */ 
body {{ margin: 0; padding: 0; min-width: 100%; width: 100% !important; height: 100% !important;}}
body, table, td, div, p, a {{ -webkit-font-smoothing: antialiased; text-size-adjust: 100%; -ms-text-size-adjust: 100%; -webkit-text-size-adjust: 100%; line-height: 100%; }}
table, td {{ mso-table-lspace: 0pt; mso-table-rspace: 0pt; border-collapse: collapse !important; border-spacing: 0; }}
img {{ border: 0; line-height: 100%; outline: none; text-decoration: none; -ms-interpolation-mode: bicubic; }}
#outlook a {{ padding: 0; }}
.ReadMsgBody {{ width: 100%; }} .ExternalClass {{ width: 100%; }}
.ExternalClass, .ExternalClass p, .ExternalClass span, .ExternalClass font, .ExternalClass td, .ExternalClass div {{ line-height: 100%; }}

/* Rounded corners for advanced mail clients only */ 
@media all and (min-width: 560px) {{
	.container {{ border-radius: 8px; -webkit-border-radius: 8px; -moz-border-radius: 8px; -khtml-border-radius: 8px;}}
}}

/* Set color for auto links (addresses, dates, etc.) */ 
a, a:hover {{
	color: #127DB3;
}}
.footer a, .footer a:hover {{
	color: #999999;
}}

 	</style>

	<!-- MESSAGE SUBJECT -->
	<title>{title}</title>

</head>

<!-- BODY -->
<!-- Set message background color (twice) and text color (twice) -->
<body topmargin=""0"" rightmargin=""0"" bottommargin=""0"" leftmargin=""0"" marginwidth=""0"" marginheight=""0"" width=""100%"" style=""border-collapse: collapse; border-spacing: 0; margin: 0; padding: 0; width: 100%; height: 100%; -webkit-font-smoothing: antialiased; text-size-adjust: 100%; -ms-text-size-adjust: 100%; -webkit-text-size-adjust: 100%; line-height: 100%;
	background-color: #F0F0F0;
	color: #000000;""
	bgcolor=""#F0F0F0""
	  dir=""rtl""
	text=""#000000"">

<!-- SECTION / BACKGROUND -->
<!-- Set message background color one again -->
<table width=""100%"" align=""center"" border=""0"" cellpadding=""0"" cellspacing=""0"" style=""border-collapse: collapse; border-spacing: 0; margin: 0; padding: 0; width: 100%;"" class=""background""><tr><td align=""center"" valign=""top"" style=""border-collapse: collapse; border-spacing: 0; margin: 0; padding: 0;""
	bgcolor=""#F0F0F0"">

<!-- WRAPPER -->
<!-- Set wrapper width (twice) -->
<table border=""0"" cellpadding=""0"" cellspacing=""0"" align=""center""
	width=""560"" style=""border-collapse: collapse; border-spacing: 0; padding: 0; width: inherit;
	max-width: 560px;"" class=""wrapper"">

	<tr>
		<td align=""center"" valign=""top"" style=""border-collapse: collapse; border-spacing: 0; margin: 0; padding: 0; padding-left: 6.25%; padding-right: 6.25%; width: 87.5%;
			padding-top: 20px;
			padding-bottom: 20px;"">

			<!-- PREHEADER -->
			<!-- Set text color to background color -->
			<div style=""display: none; visibility: hidden; overflow: hidden; opacity: 0; font-size: 1px; line-height: 1px; height: 0; max-height: 0; max-width: 0;
			color: #F0F0F0;"" class=""preheader""></div>

			<!-- LOGO -->
			<img src='cid:logo' height=""69"" width=""365"" />
		</td>
	</tr>

<!-- End of WRAPPER -->
</table>

<!-- WRAPPER / CONTEINER -->
<!-- Set conteiner background color -->
<table border=""0"" cellpadding=""0"" cellspacing=""0"" align=""center""
	bgcolor=""#FFFFFF""
	width=""560"" style=""border-collapse: collapse; border-spacing: 0; padding: 0; width: inherit;
	max-width: 560px;"" class=""container"">

	<!-- HEADER -->
	<!-- Set text color and font family (""sans-serif"" or ""Georgia, serif"") -->
	<tr>
		<td align=""center"" valign=""top"" style=""border-collapse: collapse; border-spacing: 0; margin: 0; padding: 0; padding-left: 6.25%; padding-right: 6.25%; width: 87.5%; font-size: 24px; font-weight: bold; line-height: 130%;
			padding-top: 25px;
			color: #000000;
			font-family: sans-serif;"" class=""header"">
				{title}
		</td>
	</tr>

	<tr>
		<td align=""center"" valign=""top"" style=""border-collapse: collapse; border-spacing: 0; margin: 0; padding: 0; padding-left: 6.25%; padding-right: 6.25%; width: 87.5%;
			padding-top: 25px;
			padding-bottom: 5px;"" class=""button""><a
			href=""{url}"" target=""_blank"" style=""text-decoration: underline;"">
				<table border=""0"" cellpadding=""0"" cellspacing=""0"" align=""center"" style=""max-width: 240px; min-width: 120px; border-collapse: collapse; border-spacing: 0; padding: 0;""><tr><td align=""center"" valign=""middle"" style=""padding: 12px 24px; margin: 0; text-decoration: underline; border-collapse: collapse; border-spacing: 0; border-radius: 4px; -webkit-border-radius: 4px; -moz-border-radius: 4px; -khtml-border-radius: 4px;""
					bgcolor=""#E9703E""><a target=""_blank"" style=""text-decoration: underline;
					color: #FFFFFF; font-family: sans-serif; font-size: 17px; font-weight: 400; line-height: 120%;""
					href=""{url}"">
						فتح البلاغ
					</a>
			</td></tr>

	<!-- HERO IMAGE -->
	<!-- Image text color should be opposite to background color. Set your url, image src, alt and title. Alt text should fit the image size. Real image size should be x2 (wrapper x2). Do not set height for flexible images (including ""auto""). URL format: http://domain.com/?utm_source={{{{Campaign-Source}}}}&utm_medium=email&utm_content={{{{Ìmage-Name}}}}&utm_campaign={{{{Campaign-Name}}}} -->
	<tr>
		<td align=""center"" valign=""top"" style=""border-collapse: collapse; border-spacing: 0; margin: 0; padding: 0;
			padding-top: 20px;"" class=""hero"">
			<img src='cid:emailbg' width=""560"" style=""height:200px;width: 100%;max-width: 560px;color: #000000; font-size: 13px; margin: 0; padding: 0; outline: none; text-decoration: none; -ms-interpolation-mode: bicubic; border: none; display: block;"" />
		</td>
	</tr>
</table>

<!-- End of SECTION / BACKGROUND -->
</td></tr></table>

</body>
</html>
";
            return html;
        }
