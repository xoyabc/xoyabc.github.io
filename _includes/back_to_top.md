    <!-- header.html back to top start -->
    <link rel="stylesheet" media="screen,projection" href="/assets/css/ui.totop.css" />
    <!-- header.html back to top end -->
	
	
	<!-- footer.html back to top start -->
    <!-- jquery -->
    <script src="/assets/js/jquery-1.7.2.min.js" type="text/javascript"></script>
    <!-- easing plugin ( optional ) -->
    <script src="/assets/js/easing.js" type="text/javascript"></script>
    <!-- UItoTop plugin -->
    <script src="/assets/js/jquery.ui.totop.js" type="text/javascript"></script>
    <!-- Starting the plugin -->
    <script type="text/javascript">
        $(document).ready(function() {
            /*
            var defaults = {
                containerID: 'toTop', // fading element id
                containerHoverID: 'toTopHover', // fading element hover id
                scrollSpeed: 1200,
                easingType: 'linear'
            };
            */
            $().UItoTop({ easingType: 'easeOutQuart' });
        });
    </script>
    <!-- footer.html back to top end -->
